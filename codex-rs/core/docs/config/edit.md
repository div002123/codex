# edit.rs

## 文件作用

该文件实现了配置持久化引擎，负责对 `config.toml` 文件进行原子性的编辑和更新操作。

核心功能是提供一套离散的配置变更操作，支持在不破坏现有配置结构和注释的情况下修改配置文件。使用 `toml_edit` 库来保留 TOML 文档的格式和注释。

## 主要结构体

| 结构体名称 | 说明 |
|----------|------|
| `ConfigEdit` | 配置变更操作的枚举，定义了所有支持的离散配置修改类型 |
| `ConfigDocument` | 包装了 `DocumentMut` 和配置文件的内部表示，提供高级编辑操作 |
| `ConfigEditsBuilder` | 流式 API 构建器，用于批量组装配置编辑操作并原子性地应用 |

## 主要枚举

| 枚举名称 | 说明 |
|---------|------|
| `ConfigEdit` | 支持的配置编辑操作类型（SetModel、SetNoticeHideFullAccessWarning、ReplaceMcpServers 等） |
| `Scope` | 配置作用域（Global 或 Profile） |
| `TraversalMode` | TOML 树遍历模式（Create 创建缺失的表，或 Existing 只访问现有表） |

## 主要函数/方法

### 核心函数

| 函数 | 说明 |
|-----|------|
| `apply_blocking` | 同步应用配置编辑，使用临时文件确保原子性写入 |
| `apply` | 异步应用配置编辑，在后台线程中执行阻塞操作 |

### ConfigDocument 方法

| 方法 | 说明 |
|-----|------|
| `apply` | 应用单个配置编辑操作到文档 |
| `write_profile_value` | 写入配置值到活动的 profile |
| `write_value` | 写入配置值到指定作用域 |
| `replace_mcp_servers` | 替换整个 MCP 服务器配置表 |
| `insert` | 在指定路径插入配置值 |
| `remove` | 删除指定路径的配置值 |

### ConfigEditsBuilder 方法

| 方法 | 说明 |
|-----|------|
| `new` | 创建新的编辑器构建器 |
| `with_profile` | 设置目标配置 profile |
| `set_model` | 设置模型和推理努力级别 |
| `set_hide_full_access_warning` | 设置全访问警告确认标志 |
| `set_hide_world_writable_warning` | 设置世界可写目录警告确认标志 |
| `replace_mcp_servers` | 替换 MCP 服务器配置 |
| `set_project_trusted` | 将项目标记为受信任 |
| `apply_blocking` | 同步应用所有编辑 |
| `apply` | 异步应用所有编辑 |

### 辅助函数（document_helpers）

| 函数 | 说明 |
|-----|------|
| `ensure_table_for_write` | 确保项是可写的表，必要时转换 inline table |
| `ensure_table_for_read` | 确保项是可读的表 |
| `serialize_mcp_server` | 将 MCP 服务器配置序列化为 TOML 项 |

## 与其他模块的关系

- **使用 `config/types.rs`**：操作和序列化配置类型
- **被 `config/mod.rs` 使用**：提供配置持久化功能
- **使用 `toml_edit` 库**：保留 TOML 格式和注释的编辑能力
- **与文件系统交互**：原子性地写入配置文件

## 设计特点

1. **原子性操作**：使用临时文件确保配置更新的原子性
2. **格式保留**：使用 `toml_edit` 保留注释、空白和格式
3. **批量编辑**：支持批量组装多个编辑操作并一次性应用
4. **Profile 支持**：自动处理 profile 作用域的配置路径
5. **类型安全**：通过枚举定义所有支持的编辑操作
6. **Inline Table 迁移**：自动将 inline table 转换为显式表格以保持可读性
