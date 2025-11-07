# model_provider_info.rs

## 文件作用

`model_provider_info.rs` 定义了模型提供者的注册表和配置系统。该文件管理不同AI模型提供者（如OpenAI、Azure、Ollama等）的连接信息、认证方式和通信协议。它支持内置默认提供者和用户自定义提供者，允许用户通过配置文件扩展或覆盖默认设置。

文件处理了提供者的HTTP请求构建、认证头添加、查询参数处理、重试策略配置等功能，并提供了对两种主要线协议（Responses API和Chat Completions API）的支持。

## 主要结构体列表

- `ModelProviderInfo` - 模型提供者配置结构
  - `name` - 友好的显示名称
  - `base_url` - API基础URL
  - `env_key` - API密钥的环境变量名
  - `env_key_instructions` - 获取密钥的说明
  - `experimental_bearer_token` - 实验性的直接token配置
  - `wire_api` - 使用的线协议类型
  - `query_params` - 查询参数
  - `http_headers` - HTTP头
  - `env_http_headers` - 从环境变量读取的HTTP头
  - `request_max_retries` - 请求最大重试次数
  - `stream_max_retries` - 流最大重试次数
  - `stream_idle_timeout_ms` - 流空闲超时（毫秒）
  - `requires_openai_auth` - 是否需要OpenAI认证

- `WireApi` - 线协议枚举
  - `Responses` - Responses API
  - `Chat` - Chat Completions API

## 主要函数/方法列表

### ModelProviderInfo 方法：
- `create_request_builder()` - 创建配置好的HTTP请求构建器
- `get_full_url()` - 获取完整的API URL（包括查询参数）
- `get_query_string()` - 获取查询字符串
- `is_azure_responses_endpoint()` - 检查是否为Azure Responses端点
- `apply_http_headers()` - 应用提供者特定的HTTP头
- `api_key()` - 获取API密钥（从环境变量）
- `request_max_retries()` - 获取有效的请求最大重试次数
- `stream_max_retries()` - 获取有效的流最大重试次数
- `stream_idle_timeout()` - 获取有效的流空闲超时时间

### 模块函数：
- `built_in_model_providers()` - 返回内置的默认提供者映射表
  - "openai" - OpenAI官方提供者
  - "oss" - 开源模型提供者（Ollama等）
- `create_oss_provider()` - 创建开源模型提供者（从环境变量读取配置）
- `create_oss_provider_with_base_url()` - 使用指定base_url创建开源提供者
- `matches_azure_responses_base_url()` - 检查URL是否匹配Azure模式

### 常量：
- `BUILT_IN_OSS_MODEL_PROVIDER_ID` - 内置OSS提供者ID（"oss"）
- `DEFAULT_STREAM_IDLE_TIMEOUT_MS` - 默认流空闲超时（300秒）
- `DEFAULT_STREAM_MAX_RETRIES` - 默认流重试次数（5）
- `DEFAULT_REQUEST_MAX_RETRIES` - 默认请求重试次数（4）
- `MAX_STREAM_MAX_RETRIES` - 流重试次数上限（100）
- `MAX_REQUEST_MAX_RETRIES` - 请求重试次数上限（100）
- `DEFAULT_OLLAMA_PORT` - 默认Ollama端口（11434）

## 与其他模块的关系

- **被 client 使用**: `client.rs` 使用 `ModelProviderInfo` 来创建HTTP请求和确定通信协议
- **依赖 auth**: 使用 `CodexAuth` 进行认证
- **依赖 error**: 使用 `EnvVarError` 处理环境变量相关错误
- **依赖 default_client**: 使用 `CodexHttpClient` 和 `CodexRequestBuilder`
- **配置加载**: 从 `~/.codex/config.toml` 读取用户定义的提供者
- **被 lib.rs 导出**: 作为公共API的一部分被导出
- **环境变量集成**:
  - 支持 `OPENAI_BASE_URL` 覆盖默认OpenAI端点
  - 支持 `CODEX_OSS_BASE_URL` 和 `CODEX_OSS_PORT` 配置OSS提供者
  - 支持 `OPENAI_ORGANIZATION` 和 `OPENAI_PROJECT` 环境变量
- **序列化支持**: 实现了 serde 的 Serialize/Deserialize，支持TOML配置
- **Azure特殊处理**: 检测和处理Azure Responses API的特殊行为
- **版本信息**: 自动在请求头中包含 Codex 版本号
