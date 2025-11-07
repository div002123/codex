# default_client.rs

## 文件作用

`default_client.rs` 提供 HTTP 客户端封装，用于与 AI 模型 API 通信。它包装 `reqwest` 客户端并添加 Codex 特定的功能（如 User-Agent 管理、来源追踪等）。

## 主要结构体

### CodexHttpClient
Codex HTTP 客户端封装。

**字段:**
- `inner: reqwest::Client` - 底层 reqwest 客户端

**方法:**
- `new()` - 创建客户端
- `get()` - 发送 GET 请求
- `post()` - 发送 POST 请求
- `request()` - 发送指定方法的请求

### CodexRequestBuilder
请求构建器。

**方法:**
- `header()` - 添加头部
- `json()` - 设置 JSON 正文
- `send()` - 发送请求

## 全局变量

### USER_AGENT_SUFFIX
User-Agent 后缀（全局单例）。

**用途:**
区分不同的 MCP 客户端。

## 常量

- `DEFAULT_ORIGINATOR` - 默认来源标识（`"codex_cli_rs"`）
- `CODEX_INTERNAL_ORIGINATOR_OVERRIDE_ENV_VAR` - 来源覆盖环境变量

## 与其他模块的关系

**被依赖模块:**
- `chat_completions` - 使用客户端调用 API
- `client::ModelClient` - 底层 HTTP 通信
