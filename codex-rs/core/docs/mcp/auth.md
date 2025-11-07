# mcp/auth.rs

## 文件作用

`mcp/auth.rs` 实现了 MCP 服务器认证状态的计算和管理功能。它负责检查各个 MCP 服务器的认证配置，确定是否需要 OAuth 认证、是否已配置认证凭据等。

## 主要结构体

### McpAuthStatusEntry
MCP 服务器认证状态条目。

**字段:**
- `config: McpServerConfig` - 服务器配置
- `auth_status: McpAuthStatus` - 认证状态

**功能:**
封装单个 MCP 服务器的配置和认证状态。

## 主要函数

### compute_auth_statuses()
计算所有 MCP 服务器的认证状态。

**参数:**
- `servers: I` - 服务器迭代器，元素为 `(&String, &McpServerConfig)` 元组
- `store_mode: OAuthCredentialsStoreMode` - OAuth 凭据存储模式

**返回:** `HashMap<String, McpAuthStatusEntry>` - 服务器名到认证状态的映射

**功能:**
1. 并发处理所有服务器
2. 为每个服务器调用 `compute_auth_status()`
3. 如果计算失败，记录警告并设置为 `Unsupported`
4. 返回聚合的认证状态映射

**使用场景:**
- 会话初始化时检查所有 MCP 服务器的认证状态
- 确定哪些服务器需要用户进行 OAuth 认证
- 在 UI 中显示认证状态

### compute_auth_status() (私有)
计算单个 MCP 服务器的认证状态。

**参数:**
- `server_name: &str` - 服务器名称
- `config: &McpServerConfig` - 服务器配置
- `store_mode: OAuthCredentialsStoreMode` - OAuth 凭据存储模式

**返回:** `Result<McpAuthStatus>` - 认证状态或错误

**功能:**
根据传输类型确定认证状态：

**Stdio 传输:**
- 返回 `McpAuthStatus::Unsupported`
- 本地进程不需要认证

**StreamableHttp 传输:**
- 调用 `determine_streamable_http_auth_status()`
- 检查 bearer token、HTTP headers、OAuth 配置
- 确定是否需要用户登录

**可能的认证状态:**
- `Unsupported` - 不支持认证（如 stdio）
- `NotRequired` - 不需要认证
- `Configured` - 已配置认证凭据
- `RequiresOAuth` - 需要 OAuth 认证
- `PendingAuthorization` - 等待授权

## 认证状态类型

### McpAuthStatus
认证状态枚举（来自 `codex_protocol`）。

**变体:**
- `Unsupported` - 不支持认证
- `NotRequired` - 不需要认证
- `Configured` - 已配置认证
- `RequiresOAuth` - 需要 OAuth 流程
- `PendingAuthorization` - OAuth 授权进行中

## 与其他模块的关系

**依赖模块:**
- `codex_protocol::protocol::McpAuthStatus` - 认证状态类型
- `codex_rmcp_client::OAuthCredentialsStoreMode` - OAuth 存储模式
- `codex_rmcp_client::determine_streamable_http_auth_status` - HTTP 认证状态检查
- `config::types::{McpServerConfig, McpServerTransportConfig}` - 配置类型

**被依赖模块:**
- `codex::Session` - 在会话初始化时使用
- UI/客户端 - 显示认证状态，引导用户完成 OAuth

**核心流程:**
1. 会话初始化时调用 `compute_auth_statuses()`
2. 并发检查所有配置的 MCP 服务器
3. 对于 HTTP 服务器，检查 bearer token 或 OAuth 配置
4. 返回认证状态映射
5. UI 根据状态提示用户登录或显示已配置

## 使用示例

```rust
// 计算所有服务器的认证状态
let auth_statuses = compute_auth_statuses(
    config.mcp_servers.iter(),
    OAuthCredentialsStoreMode::InMemory,
).await;

// 检查特定服务器
if let Some(entry) = auth_statuses.get("my-server") {
    match entry.auth_status {
        McpAuthStatus::RequiresOAuth => {
            // 引导用户完成 OAuth 认证
        }
        McpAuthStatus::Configured => {
            // 服务器已就绪
        }
        _ => {}
    }
}
```

## 设计说明

### 并发处理
使用 `join_all` 并发计算所有服务器的认证状态，减少启动延迟。

### 错误处理
计算失败的服务器不会导致整体失败，而是标记为 `Unsupported` 并记录警告。

### 传输类型感知
不同传输类型有不同的认证需求：
- **Stdio** - 无需认证（本地进程）
- **HTTP** - 可能需要 bearer token 或 OAuth

### OAuth 支持
支持多种 OAuth 凭据存储模式：
- `InMemory` - 内存存储（测试用）
- `Persistent` - 持久化存储（生产用）
