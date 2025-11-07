# types.rs

## 文件作用

该文件定义了配置系统中使用的核心数据类型。包含了用于定义 `Config` 结构体字段的简单结构体和枚举定义。

这个文件遵循职责分离原则，只包含简单的类型定义，不包含业务逻辑。所有配置相关的序列化和反序列化逻辑都在此处定义。

## 主要结构体

| 结构体名称 | 说明 |
|----------|------|
| `McpServerConfig` | MCP（Model Context Protocol）服务器配置，包含传输配置、启用状态、超时设置等 |
| `McpServerTransportConfig` | MCP 服务器传输层配置，支持 Stdio 和 StreamableHttp 两种传输方式 |
| `History` | 历史记录配置，控制是否将历史记录写入 `~/.codex/history.jsonl` |
| `OtelConfigToml` | OpenTelemetry 配置（从 config.toml 加载的原始配置） |
| `OtelConfig` | 应用默认值后的有效 OpenTelemetry 配置 |
| `Tui` | TUI（终端用户界面）特定的配置选项 |
| `Notice` | 通知配置，用于跟踪用户确认的各种警告提示 |
| `SandboxWorkspaceWrite` | 沙箱工作空间写入模式的配置 |
| `ShellEnvironmentPolicyToml` | Shell 环境策略配置（TOML 格式） |
| `ShellEnvironmentPolicy` | 应用后的 Shell 环境策略 |

## 主要枚举

| 枚举名称 | 说明 |
|---------|------|
| `UriBasedFileOpener` | 基于 URI 的文件打开器类型（VSCode、Cursor、Windsurf 等） |
| `HistoryPersistence` | 历史记录持久化策略（保存所有或不保存） |
| `OtelHttpProtocol` | OTEL HTTP 协议类型（Binary 或 JSON） |
| `OtelExporterKind` | OTEL 导出器类型（None、OtlpHttp、OtlpGrpc） |
| `Notifications` | 通知配置（启用/禁用或自定义列表） |
| `ShellEnvironmentPolicyInherit` | Shell 环境变量继承策略（Core、All、None） |
| `ReasoningSummaryFormat` | 推理摘要格式（None 或 Experimental） |

## 主要函数/方法

| 函数/方法 | 说明 |
|----------|------|
| `McpServerConfig::deserialize` | 自定义反序列化逻辑，处理 stdio 和 http 传输配置的区别 |
| `UriBasedFileOpener::get_scheme` | 获取文件打开器对应的 URI scheme |
| `ShellEnvironmentPolicy::from` | 从 TOML 配置转换为运行时策略 |
| `option_duration_secs::serialize/deserialize` | Duration 类型的自定义序列化/反序列化（以秒为单位） |

## 与其他模块的关系

- **被 `config/mod.rs` 使用**：提供配置系统的核心数据类型
- **被 `config/edit.rs` 使用**：配置编辑功能需要操作这些类型
- **被 `config_loader` 模块使用**：配置加载器反序列化成这些类型
- **与协议层交互**：部分类型可以转换为 `codex_app_server_protocol` 中的对应类型

## 设计特点

1. **类型安全**：使用强类型来表示配置选项，避免运行时错误
2. **序列化友好**：所有类型都实现了 Serde 的序列化/反序列化
3. **默认值支持**：为可选配置提供合理的默认值
4. **验证逻辑**：在反序列化时进行字段验证（如 MCP 服务器配置）
