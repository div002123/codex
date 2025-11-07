# handlers/mcp_resource.rs

## 文件作用

实现 MCP 资源管理工具处理器，提供资源列表、资源模板列表和资源读取功能。

## 主要结构体

### `McpResourceHandler`

MCP 资源工具的处理器，实现 `ToolHandler` trait，支持三个工具：
- `list_mcp_resources` - 列出 MCP 资源
- `list_mcp_resource_templates` - 列出 MCP 资源模板
- `read_mcp_resource` - 读取 MCP 资源

### 参数结构体

#### `ListResourcesArgs`
- `server: Option<String>` - 服务器名称（可选，未指定则列出所有服务器）
- `cursor: Option<String>` - 分页游标

#### `ListResourceTemplatesArgs`
- `server: Option<String>` - 服务器名称（可选）
- `cursor: Option<String>` - 分页游标

#### `ReadResourceArgs`
- `server: String` - 服务器名称（必需）
- `uri: String` - 资源 URI（必需）

### 响应结构体

#### `ResourceWithServer`
包装资源并添加服务器信息：
- `server: String` - 服务器名称
- `resource: Resource` - 资源详情（flatten）

#### `ResourceTemplateWithServer`
包装资源模板并添加服务器信息：
- `server: String` - 服务器名称
- `template: ResourceTemplate` - 模板详情（flatten）

#### `ListResourcesPayload`
资源列表响应：
- `server: Option<String>` - 服务器名称
- `resources: Vec<ResourceWithServer>` - 资源列表
- `next_cursor: Option<String>` - 下一页游标

#### `ListResourceTemplatesPayload`
资源模板列表响应：
- `server: Option<String>` - 服务器名称
- `resource_templates: Vec<ResourceTemplateWithServer>` - 模板列表
- `next_cursor: Option<String>` - 下一页游标

#### `ReadResourcePayload`
资源读取响应：
- `server: String` - 服务器名称
- `uri: String` - 资源 URI
- `result: ReadResourceResult` - 读取结果（flatten）

## 主要函数

### `handle()`

根据工具名称分发到相应的处理函数：
- `list_mcp_resources` → `handle_list_resources()`
- `list_mcp_resource_templates` → `handle_list_resource_templates()`
- `read_mcp_resource` → `handle_read_resource()`

### `handle_list_resources()`

列出 MCP 资源：
1. 解析参数
2. 发送工具调用开始事件
3. 执行列表操作：
   - 如果指定服务器：调用该服务器的 `list_resources()`
   - 如果未指定：调用所有服务器并合并结果
4. 序列化响应
5. 发送工具调用结束事件

### `handle_list_resource_templates()`

列出 MCP 资源模板，流程与 `handle_list_resources()` 类似。

### `handle_read_resource()`

读取指定的 MCP 资源：
1. 解析并验证参数（server 和 uri 必需）
2. 发送工具调用开始事件
3. 调用指定服务器的 `read_resource()`
4. 序列化响应
5. 发送工具调用结束事件

### 辅助函数

#### `emit_tool_call_begin()`
发送 MCP 工具调用开始事件。

#### `emit_tool_call_end()`
发送 MCP 工具调用结束事件，包含执行时间和结果。

#### `normalize_optional_string()`
规范化可选字符串（trim 并处理空字符串）。

#### `normalize_required_string()`
规范化必需字符串（trim，空则返回错误）。

## 特殊处理

### 多服务器聚合

当列出所有服务器的资源时：
- 按服务器名称排序
- 合并所有资源到单一列表
- 不支持分页游标（cursor）

### 事件跟踪

所有操作都会发出开始和结束事件：
- `McpToolCallBeginEvent` - 记录调用开始
- `McpToolCallEndEvent` - 记录调用结束、耗时和结果

## 错误处理

- 参数解析失败：返回 "failed to parse function arguments"
- 不支持的工具：返回 "unsupported MCP resource tool"
- cursor 与多服务器冲突：返回 "cursor can only be used when a server is specified"
- 必需字段缺失：返回 "{field} must be provided"
- MCP 操作失败：返回 "resources/list failed" 或 "resources/read failed"

## 与其他模块的关系

- **依赖模块**：
  - `mcp_types` - MCP 协议类型定义
  - `Session::services::mcp_connection_manager` - MCP 连接管理器

- **事件发送**：
  - `McpToolCallBeginEvent` 和 `McpToolCallEndEvent`

- **使用场景**：
  - 列出可用的 MCP 资源
  - 浏览资源模板
  - 读取资源内容（如文档、配置等）

## 测试

包含完整的测试覆盖：
- 资源序列化
- 多服务器合并和排序
- 参数解析
- 成功/失败标记
