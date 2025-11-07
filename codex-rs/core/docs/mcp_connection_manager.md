# mcp_connection_manager.rs

## 文件作用

`mcp_connection_manager.rs` 实现了 MCP（Model Context Protocol）服务器的连接管理功能。它负责启动、管理和协调多个 MCP 服务器客户端，聚合工具列表，路由工具调用，以及管理资源访问。

## 主要结构体

### McpConnectionManager
MCP 连接管理器，管理多个 MCP 客户端实例。

**字段:**
- `clients: HashMap<String, ManagedClient>` - 服务器名到客户端的映射
- `tools: HashMap<String, ToolInfo>` - 完全限定工具名到工具信息的映射
- `tool_filters: HashMap<String, ToolFilter>` - 服务器名到工具过滤器的映射

**方法:**

#### new()
创建新的连接管理器。

**参数:**
- `mcp_servers: HashMap<String, McpServerConfig>` - MCP 服务器配置映射
- `store_mode: OAuthCredentialsStoreMode` - OAuth 凭据存储模式

**返回:** `Result<(Self, ClientStartErrors)>` - 管理器实例和启动错误映射

**功能:**
1. 并发启动所有配置的 MCP 服务器
2. 验证服务器名称格式（必须匹配 `^[a-zA-Z0-9_-]+$`）
3. 处理 Stdio 和 StreamableHttp 两种传输类型
4. 初始化每个客户端（发送 `initialize` 请求）
5. 列举所有服务器的工具
6. 应用工具过滤器（enabled/disabled）
7. 生成完全限定的工具名
8. 收集启动失败的服务器错误

**错误处理:**
- 单个服务器启动失败不影响其他服务器
- 失败的服务器会在 `ClientStartErrors` 中报告
- 工具列举失败会记录警告但不中断

#### list_all_tools()
列出所有可用工具。

**返回:** `HashMap<String, Tool>` - 完全限定工具名到工具定义的映射

**功能:**
返回聚合后的工具列表，工具名格式为 `mcp__<server>__<tool>`。

#### list_all_resources()
列出所有资源。

**返回:** `HashMap<String, Vec<Resource>>` - 服务器名到资源列表的映射

**功能:**
1. 并发查询所有服务器的资源
2. 处理分页（使用 `cursor`）
3. 检测重复 cursor 防止无限循环
4. 聚合结果按服务器分组

#### list_all_resource_templates()
列出所有资源模板。

**返回:** `HashMap<String, Vec<ResourceTemplate>>` - 服务器名到资源模板列表的映射

**功能:**
类似于 `list_all_resources()`，但返回资源模板定义。

#### call_tool()
调用指定的工具。

**参数:**
- `server: &str` - 服务器名
- `tool: &str` - 工具名
- `arguments: Option<serde_json::Value>` - 工具参数

**返回:** `Result<CallToolResult>` - 工具调用结果

**功能:**
1. 检查工具是否被过滤器禁用
2. 查找目标服务器客户端
3. 使用配置的超时时间调用工具
4. 返回结果或错误

**错误情况:**
- 工具被禁用
- 服务器不存在
- 工具调用失败

#### list_resources()
列出指定服务器的资源。

**参数:**
- `server: &str` - 服务器名
- `params: Option<ListResourcesRequestParams>` - 请求参数（包括 cursor）

**返回:** `Result<ListResourcesResult>` - 资源列表结果

#### list_resource_templates()
列出指定服务器的资源模板。

**参数:**
- `server: &str` - 服务器名
- `params: Option<ListResourceTemplatesRequestParams>` - 请求参数

**返回:** `Result<ListResourceTemplatesResult>` - 资源模板列表结果

#### read_resource()
读取指定资源。

**参数:**
- `server: &str` - 服务器名
- `params: ReadResourceRequestParams` - 读取参数（包括资源 URI）

**返回:** `Result<ReadResourceResult>` - 资源内容

**功能:**
调用指定服务器读取资源内容。

#### parse_tool_name()
解析完全限定工具名。

**参数:**
- `tool_name: &str` - 完全限定工具名（如 `mcp__server__tool`）

**返回:** `Option<(String, String)>` - `(服务器名, 工具名)` 元组

**功能:**
将完全限定工具名反解析为服务器名和原始工具名。

### ManagedClient (私有)
包装 MCP 客户端及其配置。

**字段:**
- `client: Arc<RmcpClient>` - 客户端实例
- `startup_timeout: Duration` - 启动超时
- `tool_timeout: Option<Duration>` - 工具调用超时

### ToolInfo (私有)
工具信息。

**字段:**
- `server_name: String` - 服务器名
- `tool_name: String` - 原始工具名
- `tool: Tool` - 工具定义

### ToolFilter (私有)
工具过滤器。

**字段:**
- `enabled: Option<HashSet<String>>` - 启用的工具列表（None 表示全部启用）
- `disabled: HashSet<String>` - 禁用的工具列表

**方法:**
- `from_config()` - 从配置创建过滤器
- `allows()` - 检查工具是否允许使用

**过滤逻辑:**
1. 如果设置了 `enabled` 列表，工具必须在列表中
2. 工具不能在 `disabled` 列表中
3. 两个条件都满足才允许使用

## 主要函数

### qualify_tools()
生成完全限定的工具名。

**参数:**
- `tools: Vec<ToolInfo>` - 工具信息列表

**返回:** `HashMap<String, ToolInfo>` - 完全限定名到工具信息的映射

**功能:**
1. 为每个工具生成完全限定名：`mcp__<server>__<tool>`
2. 如果名称超过 64 字符，使用 SHA1 哈希截断
3. 跳过重复的工具名
4. 返回去重后的工具映射

**工具名格式:**
- 短名称: `mcp__server_name__tool_name`
- 长名称: `mcp__server_name__t...<sha1_hash>` （总长度 64）

### filter_tools()
应用工具过滤器。

**参数:**
- `tools: Vec<ToolInfo>` - 工具列表
- `filters: &HashMap<String, ToolFilter>` - 过滤器映射

**返回:** `Vec<ToolInfo>` - 过滤后的工具列表

**功能:**
对每个工具应用其服务器的过滤器，移除被禁用的工具。

### list_all_tools()
列举所有服务器的工具。

**参数:**
- `clients: &HashMap<String, ManagedClient>` - 客户端映射

**返回:** `Result<Vec<ToolInfo>>` - 聚合的工具列表

**功能:**
1. 并发查询所有服务器的工具
2. 聚合结果
3. 记录查询失败的服务器（不中断整体流程）

### resolve_bearer_token()
解析 bearer token 环境变量。

**参数:**
- `server_name: &str` - 服务器名
- `bearer_token_env_var: Option<&str>` - 环境变量名

**返回:** `Result<Option<String>>` - token 值或错误

**功能:**
1. 如果未配置环境变量，返回 None
2. 读取环境变量值
3. 验证值非空且有效（Unicode）
4. 返回 token 或错误

**错误情况:**
- 环境变量未设置
- 环境变量为空
- 环境变量包含无效 Unicode

### is_valid_mcp_server_name()
验证服务器名称格式。

**参数:**
- `server_name: &str` - 服务器名

**返回:** `bool` - 是否有效

**规则:**
- 非空
- 只包含 ASCII 字母、数字、下划线、连字符
- 匹配正则表达式 `^[a-zA-Z0-9_-]+$`

## 与其他模块的关系

**依赖模块:**
- `codex_rmcp_client::RmcpClient` - MCP 客户端实现
- `mcp_types::{Tool, Resource, ResourceTemplate, ...}` - MCP 协议类型
- `config::types::{McpServerConfig, McpServerTransportConfig}` - 配置类型

**被依赖模块:**
- `codex::Session` - 使用管理器调用 MCP 工具
- `tools::ToolRouter` - 将 MCP 工具注册到工具路由器

**核心流程:**
1. 会话初始化时创建 `McpConnectionManager`
2. 管理器启动所有配置的 MCP 服务器
3. 列举并聚合所有工具
4. 模型请求工具时，管理器路由调用到对应服务器
5. 返回工具结果给模型

## 常量

- `MCP_TOOL_NAME_DELIMITER` - 工具名分隔符（`"__"`）
- `MAX_TOOL_NAME_LENGTH` - 最大工具名长度（64）
- `DEFAULT_STARTUP_TIMEOUT` - 默认启动超时（10秒）
- `DEFAULT_TOOL_TIMEOUT` - 默认工具调用超时（60秒）

## 测试

包含单元测试验证以下功能：
- 短工具名的完全限定
- 重复工具名的跳过
- 长工具名的哈希截断
- 工具过滤器逻辑
- 启用/禁用列表的应用
- 每个服务器独立的过滤器

## 设计说明

### 并发启动
使用 `JoinSet` 并发启动所有 MCP 服务器，减少初始化时间。

### 容错性
单个服务器失败不影响其他服务器，所有错误都被收集并报告。

### 工具名命名空间
使用完全限定工具名避免不同服务器的同名工具冲突。

### 工具过滤
支持细粒度的工具启用/禁用控制，允许用户选择性使用工具。

### 超时配置
分离启动超时和工具调用超时，允许不同的时间限制。

### 传输抽象
支持多种传输类型（Stdio、StreamableHttp），统一的客户端接口。
