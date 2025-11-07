# mcp_tool_call.rs

## 文件作用

`mcp_tool_call.rs` 实现了 MCP 工具调用的处理逻辑，负责解析工具参数、调用 MCP 服务器、发送事件通知，以及处理工具调用结果。

## 主要函数

### handle_mcp_tool_call()
处理 MCP 工具调用的主函数。

**参数:**
- `sess: &Session` - 会话对象
- `turn_context: &TurnContext` - 轮次上下文
- `call_id: String` - 调用ID（用于追踪）
- `server: String` - MCP 服务器名
- `tool_name: String` - 工具名
- `arguments: String` - 工具参数（JSON字符串）

**返回:** `ResponseInputItem` - 工具调用输出（成功或失败）

**功能:**

#### 1. 解析参数
- 将 `arguments` 字符串解析为 JSON
- 空字符串视为无参数（`None`）
- 解析失败返回错误响应

#### 2. 发送开始事件
- 创建 `McpToolCallBeginEvent`
- 包含调用ID、服务器名、工具名、参数
- 通过 `notify_mcp_tool_call_event()` 发送给客户端

#### 3. 执行工具调用
- 记录开始时间
- 调用 `sess.call_tool()` 执行实际的工具调用
- 捕获并记录错误

#### 4. 发送结束事件
- 创建 `McpToolCallEndEvent`
- 包含调用ID、服务器名、工具名、执行时长、结果
- 发送给客户端

#### 5. 返回结果
- 成功：返回 `ResponseInputItem::McpToolCallOutput` 包含结果
- 失败：返回 `ResponseInputItem::FunctionCallOutput` 包含错误信息

**错误处理:**
- 参数解析错误 → 返回 `FunctionCallOutput` 包含错误信息
- 工具调用错误 → 记录警告，返回 `McpToolCallOutput` 包含错误

### notify_mcp_tool_call_event() (私有)
发送 MCP 工具调用事件。

**参数:**
- `sess: &Session` - 会话对象
- `turn_context: &TurnContext` - 轮次上下文
- `event: EventMsg` - 事件消息

**功能:**
封装事件发送逻辑，统一发送接口。

## 事件类型

### McpToolCallBeginEvent
工具调用开始事件。

**字段:**
- `call_id: String` - 调用ID
- `invocation: McpInvocation` - 调用详情

### McpToolCallEndEvent
工具调用结束事件。

**字段:**
- `call_id: String` - 调用ID
- `invocation: McpInvocation` - 调用详情
- `duration: Duration` - 执行时长
- `result: Result<CallToolResult, String>` - 调用结果

### McpInvocation
MCP 调用信息。

**字段:**
- `server: String` - 服务器名
- `tool: String` - 工具名
- `arguments: Option<Value>` - 参数

## 返回类型

### ResponseInputItem::McpToolCallOutput
MCP 工具调用输出。

**字段:**
- `call_id: String` - 调用ID
- `result: Result<CallToolResult, String>` - 调用结果

### ResponseInputItem::FunctionCallOutput
函数调用输出（用于错误情况）。

**字段:**
- `call_id: String` - 调用ID
- `output: FunctionCallOutputPayload` - 输出载荷

## 与其他模块的关系

**依赖模块:**
- `codex::Session` - 会话管理，调用 MCP 工具
- `codex::TurnContext` - 轮次上下文
- `protocol::{EventMsg, McpInvocation, McpToolCallBeginEvent, McpToolCallEndEvent}` - 事件类型
- `codex_protocol::models::{FunctionCallOutputPayload, ResponseInputItem}` - 响应类型

**被依赖模块:**
- `response_processing` - 处理模型响应时调用此函数
- `tools::ToolRouter` - 路由工具调用到此处理器

**核心流程:**
1. 模型生成工具调用请求
2. `ToolRouter` 识别为 MCP 工具
3. 调用 `handle_mcp_tool_call()`
4. 发送 `McpToolCallBegin` 事件（UI 显示加载状态）
5. 执行工具调用
6. 发送 `McpToolCallEnd` 事件（UI 更新状态）
7. 返回结果给模型
8. 模型继续生成响应

## 使用示例

```rust
// 处理模型请求的 MCP 工具调用
let result = handle_mcp_tool_call(
    &session,
    &turn_context,
    "call-123".to_string(),
    "my-server".to_string(),
    "search_files".to_string(),
    r#"{"query": "*.rs"}"#.to_string(),
).await;

// 结果将作为下一轮输入返回给模型
match result {
    ResponseInputItem::McpToolCallOutput { call_id, result } => {
        match result {
            Ok(tool_result) => println!("Success: {:?}", tool_result),
            Err(err) => println!("Error: {}", err),
        }
    }
    _ => {}
}
```

## 时间测量

使用 `Instant::now()` 和 `elapsed()` 精确测量工具执行时间：
- 开始时间在工具调用前记录
- 结束事件包含精确的执行时长
- 用于性能监控和调试

## 错误处理策略

### 参数解析错误
- 立即返回，不执行工具调用
- 使用 `FunctionCallOutput` 格式
- 设置 `success: false`

### 工具调用错误
- 记录警告日志
- 使用 `McpToolCallOutput` 格式
- 保留完整的错误信息
- 发送结束事件包含错误

## 设计说明

### 事件通知
在工具调用前后发送事件，允许 UI 实时显示：
- 加载指示器
- 进度信息
- 执行时长
- 错误信息

### 调用ID追踪
每个工具调用有唯一ID，用于：
- 匹配开始和结束事件
- 追踪异步调用
- 调试和日志关联

### 结果类型
使用不同的输出类型区分：
- `McpToolCallOutput` - MCP 工具调用结果
- `FunctionCallOutput` - 参数解析错误

### 空参数处理
支持无参数工具调用：
- 空字符串 → `None`
- 简化工具定义
- 兼容不同的模型行为
