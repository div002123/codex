# mcp/mod.rs

## 文件作用

`mcp/mod.rs` 是 MCP（Model Context Protocol）模块的入口文件，导出子模块。

## 模块结构

```rust
pub mod auth;
```

## 子模块

### auth
MCP 服务器认证状态管理模块。提供计算和查询 MCP 服务器认证状态的功能。

## 与其他模块的关系

**包含模块:**
- `mcp::auth` - 认证状态计算

**被依赖模块:**
- `codex::Session` - 使用 MCP 认证状态信息
- `mcp_connection_manager` - MCP 连接管理
