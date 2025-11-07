# profile.rs

## 文件作用

该文件定义了配置 Profile（配置文件）的数据结构。Profile 允许用户在 `config.toml` 中定义多组配置，并在它们之间快速切换。

Profile 提供了一种便捷的方式来管理不同场景下的配置选项，例如开发环境、生产环境或不同的模型配置。

## 主要结构体

| 结构体名称 | 说明 |
|----------|------|
| `ConfigProfile` | 配置 Profile 的完整定义，包含用户可以作为一个单元定义的所有配置选项 |

## ConfigProfile 字段说明

| 字段名称 | 类型 | 说明 |
|---------|------|------|
| `model` | `Option<String>` | 模型名称 |
| `model_provider` | `Option<String>` | 模型提供商的键名 |
| `approval_policy` | `Option<AskForApproval>` | 命令执行的审批策略 |
| `sandbox_mode` | `Option<SandboxMode>` | 沙箱模式 |
| `model_reasoning_effort` | `Option<ReasoningEffort>` | 模型推理努力程度 |
| `model_reasoning_summary` | `Option<ReasoningSummary>` | 模型推理摘要设置 |
| `model_verbosity` | `Option<Verbosity>` | 模型输出详细程度 |
| `chatgpt_base_url` | `Option<String>` | ChatGPT API 基础 URL |
| `experimental_instructions_file` | `Option<PathBuf>` | 实验性指令文件路径 |
| `experimental_compact_prompt_file` | `Option<PathBuf>` | 实验性紧凑提示文件路径 |
| `include_apply_patch_tool` | `Option<bool>` | 是否包含 apply_patch 工具 |
| `experimental_use_unified_exec_tool` | `Option<bool>` | 是否使用统一执行工具 |
| `experimental_use_rmcp_client` | `Option<bool>` | 是否使用实验性 RMCP 客户端 |
| `experimental_use_freeform_apply_patch` | `Option<bool>` | 是否使用自由格式的 apply_patch |
| `experimental_sandbox_command_assessment` | `Option<bool>` | 是否启用沙箱命令评估 |
| `tools_web_search` | `Option<bool>` | 是否启用网页搜索工具 |
| `tools_view_image` | `Option<bool>` | 是否启用查看图片工具 |
| `features` | `Option<FeaturesToml>` | Profile 范围内的功能开关 |

## 类型转换

| 方法 | 说明 |
|-----|------|
| `From<ConfigProfile> for codex_app_server_protocol::Profile` | 将配置 Profile 转换为协议层的 Profile 类型 |

## 与其他模块的关系

- **被 `config/mod.rs` 使用**：作为配置系统的一部分被加载和应用
- **被 `config/edit.rs` 使用**：配置编辑时需要处理 profile 作用域
- **与协议层交互**：可转换为 app-server 协议的 Profile 类型
- **使用协议类型**：引用 `codex_protocol::config_types` 中的枚举类型

## 设计特点

1. **全部可选**：所有字段都是 `Option` 类型，允许部分覆盖
2. **继承机制**：Profile 中未设置的选项会回退到全局配置
3. **场景化配置**：支持为不同使用场景定义独立的配置集
4. **协议兼容**：可以转换为 app-server 协议格式

## 使用示例

用户可以在 `config.toml` 中定义多个 profile：

```toml
# 默认使用的 profile
profile = "fast"

[profiles.fast]
model = "gpt-4o"
sandbox_mode = "read-only"

[profiles.powerful]
model = "gpt-5-codex"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"
```

通过切换 `profile` 字段或使用命令行参数，可以快速在不同配置之间切换。
