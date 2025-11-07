# runtimes/mod.rs

## 文件作用

runtimes 模块的入口文件，提供工具运行时（ToolRuntime）的具体实现。每个运行时保持小而专注，通过编排器（Orchestrator）复用批准、沙箱和重试机制。

## 模块说明

运行时模块负责实际执行工具操作，与 handlers 模块配合工作：
- **handlers**：接收工具调用请求，处理参数解析和验证
- **runtimes**：执行实际的工具操作（如运行命令、应用补丁）

## 主要模块

### `apply_patch`
应用补丁运行时，执行已验证的补丁操作。

### `shell`
Shell 命令运行时，在沙箱环境中执行 Shell 命令。

### `unified_exec`
统一执行运行时，处理持久化的交互式 Shell 会话。

## 辅助函数

### `build_command_spec()`

构建 `CommandSpec` 的共享辅助函数。

**参数**：
- `command: &[String]` - 命令行参数数组
- `cwd: &Path` - 工作目录
- `env: &HashMap<String, String>` - 环境变量
- `timeout_ms: Option<u64>` - 超时时间（毫秒）
- `with_escalated_permissions: Option<bool>` - 是否提升权限
- `justification: Option<String>` - 权限提升理由

**返回值**：
- `Ok(CommandSpec)` - 构建成功
- `Err(ToolError::Rejected)` - 命令参数为空

**实现**：
1. 验证命令参数非空
2. 分离程序名和参数
3. 构建 `CommandSpec` 结构

## 设计模式

### 关注点分离

每个运行时专注于单一职责：
- **apply_patch**：应用代码补丁
- **shell**：执行 Shell 命令
- **unified_exec**：管理持久会话

### 编排器模式

运行时与 `ToolOrchestrator` 配合，由编排器统一处理：
- 批准流程（Approval）
- 沙箱管理（Sandbox）
- 错误重试（Retry）
- 事件发送（Events）

### Trait 抽象

每个运行时实现以下 traits：
- `ToolRuntime<Request, Response>` - 核心执行逻辑
- `Sandboxable` - 沙箱行为配置
- `Approvable<Request>` - 批准逻辑
- `ProvidesSandboxRetryData` - 重试数据提供

## 与其他模块的关系

- **上游依赖**：
  - `tools::handlers` - 处理器调用运行时
  - `tools::orchestrator` - 编排器管理运行时

- **下游依赖**：
  - `sandboxing` - 沙箱执行环境
  - `exec` - 命令执行基础设施

- **共享组件**：
  - `CommandSpec` - 命令规范
  - `ToolError` - 错误类型
  - `SandboxAttempt` - 沙箱尝试

## 执行流程

```
Handler 接收请求
    ↓
创建 Request 对象
    ↓
初始化 Runtime
    ↓
Orchestrator::run()
    ↓ 批准检查
    ↓ 沙箱设置
    ↓ Runtime::run()
    ↓ 执行操作
    ↓ 错误处理/重试
    ↓
返回 Response
    ↓
Handler 格式化输出
```

## 设计优势

- **模块化**：每个运行时独立，易于测试和维护
- **可复用**：通用逻辑（批准、沙箱）集中在编排器
- **可扩展**：添加新工具只需实现对应 traits
- **类型安全**：泛型确保请求和响应类型匹配
- **一致性**：所有工具遵循相同的执行模式
