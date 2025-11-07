# chat_completions.rs

## 文件作用

`chat_completions.rs` 实现了经典的 Chat Completions API 流式调用功能，负责将 Codex 内部的提示格式转换为标准的聊天消息格式，并处理推理内容的附加。

## 主要函数

### stream_chat_completions()
使用 Chat Completions API 流式调用模型。

**参数:**
- `prompt: &Prompt` - 提示内容
- `model_family: &ModelFamily` - 模型家族
- `client: &CodexHttpClient` - HTTP 客户端
- `provider: &ModelProviderInfo` - 模型提供商信息
- `otel_event_manager: &OtelEventManager` - OpenTelemetry 事件管理器
- `session_source: &SessionSource` - 会话来源

**返回:** `Result<ResponseStream>` - 响应流

**功能:**

#### 1. 消息转换
- 添加系统消息（包含完整指令）
- 转换 ResponseItem 为标准消息格式
- 处理文本和图片内容
- 过滤重复的助手消息

#### 2. 推理内容附加
- 将 Reasoning 块附加到相邻的助手消息
- 优先附加到前一条助手消息（stop turns）
- 否则附加到下一条助手锚点（工具调用或助手消息）
- 如果对话以用户消息结束，丢弃所有推理内容

#### 3. 多模态支持
- 用户消息支持图片（以内容项数组形式发送）
- 助手消息始终使用纯文本（兼容性）

#### 4. 工具调用处理
- 转换 FunctionCall 为工具调用格式
- 转换 FunctionCallOutput 为工具响应
- 处理本地 shell 调用

## 设计说明

### 推理内容管理
推理块不直接发送给模型，而是附加到相邻的助手消息中，保持对话流的连贯性。

### 消息去重
跟踪最后的助手消息文本，避免发送重复内容。

### 兼容性
- 不支持 output_schema（抛出错误）
- 助手消息使用纯文本格式
- 工具调用使用标准格式

## 与其他模块的关系

**依赖模块:**
- `client_common::Prompt` - 提示定义
- `default_client::CodexHttpClient` - HTTP 客户端
- `tools::spec::create_tools_json_for_chat_completions_api` - 工具规范生成

**被依赖模块:**
- `client::ModelClient` - 使用此函数调用OpenAI兼容API
