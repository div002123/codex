# handlers/mcp.rs

## 文件作用

实现 MCP（Model Context Protocol）工具调用处理器，作为连接 AI 模型和 MCP 服务器的桥梁。

## 主要结构体

### `McpHandler`

MCP 工具调用处理器，实现 `ToolHandler` trait。

**特性**：
- 工具类型：`ToolKind::Mcp`
- 仅支持 `ToolPayload::Mcp` 载荷类型

## 主要函数

### `kind()`

返回工具类型 `ToolKind::Mcp`。

### `handle()`

异步处理 MCP 工具调用：

**处理流程**：
1. 提取载荷中的服务器名、工具名和参数
2. 调用 `handle_mcp_tool_call()` 执行实际的 MCP 调用
3. 根据响应类型转换为相应的 `ToolOutput`：
   - `McpToolCallOutput` → `ToolOutput::Mcp`
   - `FunctionCallOutput` → `ToolOutput::Function`

**参数**：
- `server: String` - MCP 服务器名称
- `tool: String` - MCP 工具名称
- `raw_arguments: String` - JSON 格式的参数

**返回值**：
- 成功：`ToolOutput::Mcp { result }` 或 `ToolOutput::Function { content, content_items, success }`
- 失败：`FunctionCallError::RespondToModel` 错误

## 错误处理

- 不支持的载荷类型：返回 "mcp handler received unsupported payload"
- 意外的响应变体：返回 "mcp handler received unexpected response variant"

## 与其他模块的关系

- **依赖模块**：
  - `mcp_tool_call::handle_mcp_tool_call()` - 执行实际的 MCP 工具调用
  - `codex_protocol::models::ResponseInputItem` - 响应类型定义

- **调用流程**：
  ```
  AI 模型 → McpHandler → handle_mcp_tool_call() → MCP 服务器
                ↓
           转换响应
                ↓
          返回给 AI 模型
  ```

- **协作模块**：
  - `ToolInvocation` - 工具调用上下文
  - `Session` 和 `TurnContext` - 会话和轮次上下文

- **使用场景**：
  - AI 模型调用外部 MCP 服务器提供的工具
  - 例如：文件系统访问、数据库查询、API 调用等

## 设计特点

- **简洁性**：作为薄层代理，主要职责是协议转换
- **灵活性**：支持任意 MCP 工具调用
- **类型安全**：通过模式匹配确保响应类型正确
