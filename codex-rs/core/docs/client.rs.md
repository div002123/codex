# client.rs

## 文件作用

`client.rs` 实现了与AI模型API进行通信的核心客户端逻辑。该文件包含 `ModelClient` 结构体，负责处理HTTP请求、流式响应解析、错误处理和重试逻辑。它支持两种线协议：OpenAI Responses API 和 Chat Completions API，并实现了SSE（Server-Sent Events）事件流的解析和处理。

文件还包含了复杂的错误处理机制，包括速率限制、使用额度限制、认证刷新等场景的处理，以及自动重试和退避策略。

## 主要结构体列表

- `ModelClient` - 模型客户端，管理与AI模型的通信
- `ErrorResponse` - API错误响应
- `Error` - 错误详情结构
- `SseEvent` - SSE事件
- `ResponseCompleted` - 响应完成事件
- `ResponseCompletedUsage` - Token使用统计
- `ResponseCompletedInputTokensDetails` - 输入token详情
- `ResponseCompletedOutputTokensDetails` - 输出token详情
- `StreamAttemptError` - 流尝试错误（枚举）

## 主要函数/方法列表

### ModelClient 方法：
- `new()` - 创建新的客户端实例
- `stream()` - 开始流式请求
- `stream_responses()` - 使用 Responses API 进行流式请求
- `attempt_stream_responses()` - 单次流式请求尝试
- `get_model_context_window()` - 获取模型上下文窗口大小
- `get_auto_compact_token_limit()` - 获取自动压缩token限制
- `config()` - 获取配置
- `provider()` - 获取提供者信息
- `get_provider()` - 获取提供者（克隆）
- `get_otel_event_manager()` - 获取遥测事件管理器
- `get_session_source()` - 获取会话来源
- `get_model()` - 获取模型名称
- `get_model_family()` - 获取模型家族
- `get_reasoning_effort()` - 获取推理努力程度
- `get_reasoning_summary()` - 获取推理摘要设置
- `get_auth_manager()` - 获取认证管理器

### 辅助函数：
- `process_sse()` - 处理SSE事件流
- `stream_from_fixture()` - 从测试fixture流式读取
- `attach_item_ids()` - 为Azure添加item ID
- `parse_rate_limit_snapshot()` - 解析速率限制快照
- `parse_rate_limit_window()` - 解析速率限制窗口
- `parse_header_f64()` / `parse_header_i64()` / `parse_header_str()` - 解析HTTP头
- `try_parse_retry_after()` - 解析重试延迟
- `is_context_window_error()` - 检查是否为上下文窗口错误
- `is_quota_exceeded_error()` - 检查是否为配额超限错误
- `rate_limit_regex()` - 获取速率限制正则表达式

## 与其他模块的关系

- **依赖 client_common**: 使用 `Prompt`、`ResponseEvent`、`ResponseStream` 等通用类型
- **依赖 model_provider_info**: 使用 `ModelProviderInfo` 和 `WireApi` 来确定如何与提供者通信
- **依赖 model_family**: 使用 `ModelFamily` 来确定模型特定的行为
- **依赖 error**: 使用和返回 `CodexErr` 及其变体
- **依赖 auth**: 使用 `AuthManager` 和 `CodexAuth` 进行认证
- **依赖 config**: 使用 `Config` 获取配置信息
- **依赖 chat_completions**: 当使用 Chat API 时，委托给 `stream_chat_completions`
- **依赖 openai_model_info**: 获取模型元数据（上下文窗口等）
- **依赖 token_data**: 解析plan类型和使用限制信息
- **依赖 default_client**: 使用 `CodexHttpClient` 进行HTTP通信
- **使用 codex-otel**: 记录遥测事件
- **使用 codex-protocol**: 使用协议类型如 `TokenUsage`、`RateLimitSnapshot` 等
