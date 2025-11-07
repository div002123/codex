# error.rs

## 文件作用

`error.rs` 定义了整个 Codex 核心库的错误类型系统。该文件使用 `thiserror` crate 定义了结构化的错误类型层次，涵盖了从网络通信、认证、沙箱执行到使用限制等各种错误场景。它提供了友好的错误消息格式化功能，包括针对不同订阅计划类型的定制化错误提示，以及重试时间戳的本地化格式化。

文件还包含了沙箱错误、环境变量错误等专用错误类型，以及将错误转换为用户界面友好消息的工具函数。

## 主要结构体列表

### 错误类型：
- `CodexErr` - 主错误枚举，包含所有可能的错误类型
- `SandboxErr` - 沙箱执行错误（枚举）
  - Denied - 沙箱拒绝执行
  - SeccompInstall - Seccomp设置错误（Linux）
  - SeccompBackend - Seccomp后端错误（Linux）
  - Timeout - 命令超时
  - Signal - 被信号杀死
  - LandlockRestrict - Landlock限制错误

### 错误详情结构：
- `ConnectionFailedError` - 连接失败错误
- `ResponseStreamFailed` - 响应流失败错误
- `RefreshTokenFailedError` - Token刷新失败错误
- `RefreshTokenFailedReason` - Token刷新失败原因（枚举）
- `UnexpectedResponseError` - 意外响应错误
- `RetryLimitReachedError` - 重试限制达到错误
- `UsageLimitReachedError` - 使用限制达到错误
- `EnvVarError` - 环境变量错误

### 类型别名：
- `Result<T>` - 标准Result类型，错误为 `CodexErr`

## 主要函数/方法列表

### CodexErr 方法：
- `downcast_ref()` - 向下转型为具体错误类型

### UsageLimitReachedError 格式化：
- `fmt()` - 实现Display trait，根据plan类型生成定制化错误消息

### 辅助函数：
- `get_error_message_ui()` - 将错误转换为用户界面友好的消息
- `retry_suffix()` - 生成重试时间后缀
- `retry_suffix_after_or()` - 生成"或者在...重试"后缀
- `format_retry_timestamp()` - 格式化重试时间戳（本地化）
- `day_suffix()` - 获取日期序数后缀（st, nd, rd, th）
- `now_for_retry()` - 获取当前时间（支持测试覆盖）

### Display实现：
- 所有错误类型都实现了 `Display` trait，提供人类可读的错误消息

## 与其他模块的关系

- **核心错误类型**: 被整个 codex-core 库使用，定义了 `Result<T>` 类型别名
- **依赖 exec**: 使用 `ExecToolCallOutput`、`StreamOutput` 等执行相关类型
- **依赖 token_data**: 使用 `PlanType`、`KnownPlan` 来生成plan特定的错误消息
- **依赖 codex**: 使用 `ProcessedResponseItem`
- **使用 codex-protocol**: 使用 `ConversationId`、`RateLimitSnapshot` 等协议类型
- **使用 codex-async-utils**: 从 `CancelErr` 转换为 `CodexErr`
- **被 client 使用**: `client.rs` 返回和处理这些错误类型
- **沙箱集成**: 定义了 `SandboxErr`，被沙箱模块使用
- **平台特定**: 包含Linux特定的landlock和seccomp错误
- **测试支持**: 包含测试辅助功能（如 `NOW_OVERRIDE`）用于时间相关测试
- **错误转换**: 实现了从标准库和第三方库错误类型的自动转换（From trait）
- **用户体验**: 提供了针对不同订阅计划的定制化错误消息和帮助链接
