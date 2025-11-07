# unified_exec/errors.rs

## 文件作用

定义统一执行模块的错误类型，使用 `thiserror` 提供清晰的错误消息。

## 错误类型

### UnifiedExecError
```rust
pub(crate) enum UnifiedExecError {
    #[error("Failed to create unified exec session: {message}")]
    CreateSession { message: String },

    #[error("Unknown session id {session_id}")]
    UnknownSessionId { session_id: i32 },

    #[error("failed to write to stdin")]
    WriteToStdin,

    #[error("missing command line for unified exec request")]
    MissingCommandLine,

    #[error("Command denied by sandbox: {message}")]
    SandboxDenied {
        message: String,
        output: ExecToolCallOutput,
    },
}
```

## 错误变体说明

### CreateSession
创建统一执行会话失败。

**使用场景**：
- PTY 进程生成失败
- 编排器返回错误
- 沙箱转换失败

**构造方法**：
```rust
pub(crate) fn create_session(message: String) -> Self
```

### UnknownSessionId
尝试访问不存在的会话 ID。

**使用场景**：
- `write_stdin` 请求中指定了无效的 session_id
- 会话已完成并被清理

### WriteToStdin
向会话的标准输入写入失败。

**使用场景**：
- PTY 会话的写入通道已关闭
- 会话进程已终止

### MissingCommandLine
执行请求中缺少命令行。

**使用场景**：
- `ExecEnv` 的 command 向量为空
- 命令解析失败

### SandboxDenied
命令被沙箱拒绝。

**包含信息**：
- `message`：拒绝原因的简短描述（从输出中截取）
- `output`：完整的执行输出（用于调试和日志）

**构造方法**：
```rust
pub(crate) fn sandbox_denied(message: String, output: ExecToolCallOutput) -> Self
```

**使用场景**：
- 会话在沙箱模式下运行并提前退出
- 输出包含沙箱拒绝关键词
- 退出码指示权限被拒绝

## 错误处理策略

1. **CreateSession**：通常向上传播，导致工具调用失败
2. **UnknownSessionId**：返回给客户端，提示会话无效
3. **WriteToStdin**：返回给客户端，提示写入失败
4. **MissingCommandLine**：在早期验证阶段捕获
5. **SandboxDenied**：触发重试逻辑（如果允许），使用无沙箱模式

## 与其他模块的关系

- **mod.rs**：使用所有错误类型
- **session.rs**：返回 `SandboxDenied` 和 `CreateSession`
- **session_manager.rs**：返回 `UnknownSessionId`、`WriteToStdin` 和 `MissingCommandLine`
- **exec.rs**：使用 `ExecToolCallOutput` 结构
- **tools/orchestrator**：处理 `CreateSession` 和 `SandboxDenied` 以实现重试逻辑
