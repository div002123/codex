# runtimes/unified_exec.rs

## 文件作用

实现统一执行（Unified Exec）运行时，在编排器（Orchestrator）下处理持久化 PTY（伪终端）会话的批准和沙箱编排，委托给会话管理器执行实际操作。

## 主要结构体

### `UnifiedExecRequest`

统一执行请求：
- `command: Vec<String>` - 命令行参数数组
- `cwd: PathBuf` - 工作目录
- `env: HashMap<String, String>` - 环境变量

**Trait 实现**：
- `ProvidesSandboxRetryData`：提供重试所需的命令和工作目录

**构造函数**：
```rust
pub fn new(command: Vec<String>, cwd: PathBuf, env: HashMap<String, String>) -> Self
```

### `UnifiedExecRuntime<'a>`

统一执行运行时，持有对 `UnifiedExecSessionManager` 的引用。

**生命周期**：
- `'a`：绑定到 `UnifiedExecSessionManager` 的生命周期

**Trait 实现**：
- `ToolRuntime<UnifiedExecRequest, UnifiedExecSession>`
- `Sandboxable`
- `Approvable<UnifiedExecRequest>`

**构造函数**：
```rust
pub fn new(manager: &'a UnifiedExecSessionManager) -> Self
```

### `UnifiedExecApprovalKey`

批准缓存键：
- `command: Vec<String>` - 命令
- `cwd: PathBuf` - 工作目录

用于批准缓存，避免重复批准相同的命令。

## Trait 实现

### `Sandboxable`

沙箱行为配置：
- `sandbox_preference()` → `Auto`：自动选择沙箱策略
- `escalate_on_failure()` → `true`：失败时升级到无沙箱模式

### `Approvable<UnifiedExecRequest>`

批准逻辑配置。

#### `approval_key()`

返回批准缓存键（command + cwd）。

#### `start_approval_async()`

异步启动批准流程：
1. 构建批准键
2. 使用 `with_cached_approval()` 检查缓存
3. 调用 `Session::request_command_approval()` 请求批准
4. 传递重试原因（如果有）和风险评估

**批准缓存**：
相同的命令和工作目录只需批准一次。

### `ToolRuntime<UnifiedExecRequest, UnifiedExecSession>`

#### `run()`

执行统一执行请求：

**处理流程**：
1. **构建命令规范**：
   ```rust
   let spec = build_command_spec(&req.command, &req.cwd, &req.env, None, None, None)?;
   ```
   - 无超时（PTY 会话可以长期运行）
   - 无权限提升（由沙箱决定）
   - 无理由（由批准流程处理）

2. **获取执行环境**：
   ```rust
   let exec_env = attempt.env_for(&spec)?;
   ```
   根据当前沙箱尝试创建执行环境

3. **打开 PTY 会话**：
   ```rust
   self.manager.open_session_with_exec_env(&exec_env).await?;
   ```
   委托给会话管理器创建 PTY 会话

4. **错误处理**：
   - `SandboxDenied`：转换为 `ToolError::Codex(SandboxErr::Denied)`
   - 其他错误：转换为 `ToolError::Rejected`

**返回值**：
- `Ok(UnifiedExecSession)` - 成功创建的 PTY 会话
- `Err(ToolError)` - 执行失败

## 执行流程

```
UnifiedExecHandler::exec_command()
    ↓
创建 UnifiedExecRequest
    ↓
ToolOrchestrator::run()
    ↓
检查批准缓存
    ↓ 未缓存
start_approval_async()
    ↓ 批准通过
创建 SandboxAttempt
    ↓
UnifiedExecRuntime::run()
    ↓
build_command_spec()
    ↓
attempt.env_for(&spec)
    ↓
manager.open_session_with_exec_env()
    ↓
返回 UnifiedExecSession
    ↓
Session Manager 管理 PTY
    ↓
Handler 收集输出
    ↓
返回给 AI 模型
```

## 与 Shell Runtime 的对比

| 特性 | ShellRuntime | UnifiedExecRuntime |
|------|-------------|-------------------|
| 执行模型 | 一次性命令 | 持久化会话 |
| 输出方式 | 同步收集 | 流式输出 |
| 交互性 | 不支持 | 支持 stdin 输入 |
| 会话管理 | 无 | UnifiedExecSessionManager |
| 返回值 | ExecToolCallOutput | UnifiedExecSession |
| 使用场景 | 快速命令 | REPL、调试器、长期任务 |

## 沙箱拒绝处理

当沙箱拒绝执行时：
```rust
UnifiedExecError::SandboxDenied { output, .. } => {
    ToolError::Codex(CodexErr::Sandbox(SandboxErr::Denied {
        output: Box::new(output),
    }))
}
```

**重要性**：
- 保留沙箱的拒绝输出
- 允许编排器识别沙箱问题
- 可能触发重试或升级流程

## 与其他模块的关系

- **调用者**：
  - `handlers::unified_exec::UnifiedExecHandler`

- **依赖模块**：
  - `unified_exec::UnifiedExecSessionManager` - 会话管理器
  - `unified_exec::UnifiedExecSession` - PTY 会话
  - `unified_exec::UnifiedExecError` - 错误类型
  - `tools::runtimes::build_command_spec()` - 构建命令规范

- **批准系统**：
  - `Session::request_command_approval()` - 请求批准
  - `with_cached_approval()` - 批准缓存

- **沙箱系统**：
  - `SandboxAttempt::env_for()` - 创建执行环境
  - `SandboxErr::Denied` - 沙箱拒绝错误

## 会话生命周期

```
创建请求
    ↓
批准通过
    ↓
open_session_with_exec_env()
    ↓
创建 PTY
    ↓
分配 session_id
    ↓
返回 UnifiedExecSession
    ↓
Handler 执行命令
    ↓
收集输出
    ↓
返回 session_id（如果仍在运行）
    ↓
AI 模型使用 session_id
    ↓
write_stdin(session_id, input)
    ↓
继续交互...
    ↓
进程退出或超时
    ↓
会话清理
```

## 错误处理

- 命令参数为空：返回 `ToolError::Rejected("missing command line for PTY")`
- 沙箱环境创建失败：返回 `ToolError::Codex`
- 沙箱拒绝：返回 `ToolError::Codex(SandboxErr::Denied)`
- 会话创建失败：返回 `ToolError::Rejected`

## 设计特点

- **委托模式**：运行时只负责编排，实际执行委托给会话管理器
- **批准缓存**：避免重复批准相同命令
- **沙箱集成**：完全集成到沙箱系统
- **错误透传**：保留沙箱拒绝的详细信息
- **生命周期管理**：通过引用绑定到会话管理器

## 使用场景

### 交互式 REPL

```rust
// 启动 Python
UnifiedExecRequest {
    command: vec!["python3".to_string()],
    cwd: PathBuf::from("/workspace"),
    env: HashMap::new(),
}
```

### 长期运行任务

```rust
// 启动开发服务器
UnifiedExecRequest {
    command: vec!["npm".to_string(), "run".to_string(), "dev".to_string()],
    cwd: PathBuf::from("/workspace/app"),
    env: env_with_node_path,
}
```

### 调试会话

```rust
// 启动 GDB
UnifiedExecRequest {
    command: vec!["gdb".to_string(), "./app".to_string()],
    cwd: PathBuf::from("/workspace/build"),
    env: HashMap::new(),
}
```

## 安全考虑

- **批准要求**：所有 PTY 会话都需要批准
- **沙箱执行**：在沙箱环境中运行，受策略限制
- **会话隔离**：每个会话独立，不共享状态
- **资源限制**：可以设置超时和资源限制（由管理器处理）
