# runtimes/shell.rs

## 文件作用

实现 Shell 命令运行时，在编排器（Orchestrator）下执行 Shell 命令，支持批准流程、沙箱管理和命令安全性检查。

## 主要结构体

### `ShellRequest`

Shell 命令请求：
- `command: Vec<String>` - 命令行参数数组
- `cwd: PathBuf` - 工作目录
- `timeout_ms: Option<u64>` - 超时时间（毫秒）
- `env: HashMap<String, String>` - 环境变量
- `with_escalated_permissions: Option<bool>` - 是否请求权限提升
- `justification: Option<String>` - 权限提升理由

**Trait 实现**：
- `ProvidesSandboxRetryData`：提供重试所需的命令和工作目录

### `ShellRuntime`

Shell 命令运行时。

**Trait 实现**：
- `ToolRuntime<ShellRequest, ExecToolCallOutput>`
- `Sandboxable`
- `Approvable<ShellRequest>`

### `ApprovalKey`

批准缓存键：
- `command: Vec<String>` - 命令
- `cwd: PathBuf` - 工作目录
- `escalated: bool` - 是否提升权限

用于批准缓存，避免重复批准相同的命令。

## 主要函数

### `stdout_stream()`

创建标准输出流，用于实时输出命令执行过程。

## Trait 实现

### `Sandboxable`

沙箱行为配置：
- `sandbox_preference()` → `Auto`：自动选择沙箱策略
- `escalate_on_failure()` → `true`：失败时升级到无沙箱模式

### `Approvable<ShellRequest>`

批准逻辑配置，包含复杂的安全性判断。

#### `approval_key()`

返回批准缓存键（command + cwd + escalated）。

#### `start_approval_async()`

异步启动批准流程：
1. 构建批准键
2. 确定批准原因：
   - 优先使用重试原因（retry_reason）
   - 否则使用权限提升理由（justification）
3. 使用批准缓存调用 `request_command_approval()`

**批准缓存**：
相同的命令和上下文只需批准一次。

#### `wants_initial_approval()`

确定命令是否需要初始批准，基于复杂的安全策略：

**安全命令检查**：
```rust
if is_known_safe_command(&req.command) {
    return false; // 已知安全命令不需要批准
}
```

**按批准策略处理**：

1. **`AskForApproval::Never`** 或 **`AskForApproval::OnFailure`**：
   - 不需要初始批准
   - 只在失败时批准

2. **`AskForApproval::OnRequest`**：

   根据沙箱策略细分：

   - **`SandboxPolicy::DangerFullAccess`**：
     ```rust
     return command_might_be_dangerous(&req.command);
     ```
     只对危险命令提示

   - **其他沙箱（ReadOnly/WorkspaceWrite）**：
     ```rust
     if req.with_escalated_permissions.unwrap_or(false) {
         return true; // 权限提升必须批准
     }
     return command_might_be_dangerous(&req.command);
     ```
     对于非提升、非危险命令，让沙箱静默执行

3. **`AskForApproval::UnlessTrusted`**：
   ```rust
   return !is_known_safe_command(&req.command);
   ```
   除非是已知安全命令，否则都需要批准

**设计理念**：
- 在受限沙箱中，非危险命令可以静默运行
- 沙箱会强制执行限制（如阻止网络、写入）
- 减少不必要的用户提示，提升体验

#### `wants_escalated_first_attempt()`

确定是否首次尝试就使用权限提升：
```rust
req.with_escalated_permissions.unwrap_or(false)
```

如果用户明确请求权限提升，立即使用。

### `ToolRuntime<ShellRequest, ExecToolCallOutput>`

#### `run()`

执行 Shell 命令：
1. 调用 `build_command_spec()` 构建命令规范
2. 从 `SandboxAttempt` 获取执行环境
3. 使用 `execute_env()` 执行命令
4. 返回执行结果

**参数**：
- `req: &ShellRequest` - Shell 请求
- `attempt: &SandboxAttempt<'_>` - 沙箱尝试上下文
- `ctx: &ToolCtx<'_>` - 工具上下文

## 命令安全性分类

### 已知安全命令

通过 `is_known_safe_command()` 识别：
- `ls`、`pwd`、`echo`
- `git status`、`git log`
- `cat`、`grep`（只读操作）
- 等等

**特点**：
- 不修改系统状态
- 不需要网络访问
- 不需要特殊权限

### 可能危险命令

通过 `command_might_be_dangerous()` 识别：
- `rm`、`mv`（可能删除文件）
- `curl`、`wget`（网络访问）
- `sudo`、`su`（权限提升）
- `chmod`、`chown`（权限修改）
- 等等

**特点**：
- 可能造成数据丢失
- 可能访问外部资源
- 可能需要特殊权限

## 批准策略详解

### DangerFullAccess 模式

在此模式下，系统没有沙箱限制，因此：
- **安全命令**：静默执行
- **危险命令**：提示批准

示例：
```bash
ls -la          # 静默执行
rm -rf /tmp/*   # 提示批准
```

### 受限沙箱模式（ReadOnly/WorkspaceWrite）

在此模式下，沙箱会强制执行限制，因此：
- **非提升 + 非危险命令**：静默执行（让沙箱处理）
- **提升权限命令**：提示批准
- **危险命令**：提示批准

示例：
```bash
npm install     # 静默执行，沙箱会阻止写入（如果是 ReadOnly）
curl example.com # 静默执行，沙箱会阻止网络
sudo apt update  # 提示批准（权限提升）
```

## 执行流程

```
ShellHandler
    ↓
创建 ShellRequest
    ↓
ToolOrchestrator::run()
    ↓
wants_initial_approval()
    ↓ 需要批准
start_approval_async()
    ↓ 批准通过
创建 SandboxAttempt
    ↓
ShellRuntime::run()
    ↓
build_command_spec()
    ↓
execute_env()
    ↓
返回 ExecToolCallOutput
    ↓
格式化响应
    ↓
返回给 AI 模型
```

## 与其他模块的关系

- **调用者**：
  - `handlers::shell::ShellHandler`

- **依赖模块**：
  - `command_safety::is_dangerous_command` - 危险命令检测
  - `command_safety::is_safe_command` - 安全命令检测
  - `sandboxing::execute_env()` - 执行环境
  - `tools::runtimes::build_command_spec()` - 构建命令规范

- **批准系统**：
  - `Session::request_command_approval()` - 请求批准
  - `with_cached_approval()` - 批准缓存

## 错误处理

- 命令参数为空：返回 `ToolError::Rejected`
- 沙箱环境创建失败：返回 `ToolError::Codex`
- 命令执行失败：返回 `ToolError::Codex`

## 设计特点

- **智能批准**：基于命令安全性和沙箱策略动态决策
- **批准缓存**：避免重复批准相同命令
- **沙箱分层**：在不同沙箱级别使用不同策略
- **用户体验**：减少不必要的提示，提升流畅度
- **安全性**：危险操作始终需要用户确认
- **灵活性**：支持权限提升和自定义理由

## 批准决策示例

| 命令 | 沙箱 | 策略 | 权限提升 | 是否批准 | 原因 |
|------|------|------|---------|---------|------|
| `ls -la` | Any | Any | No | ✗ | 已知安全 |
| `rm -rf /` | DangerFullAccess | OnRequest | No | ✓ | 危险命令 |
| `npm install` | ReadOnly | OnRequest | No | ✗ | 让沙箱阻止 |
| `npm install` | DangerFullAccess | OnRequest | No | ✗ | 不危险 |
| `sudo apt update` | Any | OnRequest | Yes | ✓ | 权限提升 |
| `curl api.com` | WorkspaceWrite | OnRequest | No | ✓ | 危险（网络） |
