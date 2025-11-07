# mod.rs

## 文件作用

该文件是配置系统的核心模块，负责加载、解析和管理应用程序的所有配置。它整合了用户配置、命令行覆盖、Profile 配置以及各种默认值，生成最终的运行时配置。

配置系统支持分层配置（用户配置、托管配置、托管首选项）、Profile 机制、项目级别的信任设置，以及广泛的命令行覆盖选项。

## 主要结构体

| 结构体名称 | 说明 |
|----------|------|
| `Config` | 应用程序的最终配置，包含所有运行时设置和已解析的选项 |
| `ConfigToml` | 从 `~/.codex/config.toml` 反序列化的基础配置 |
| `ConfigOverrides` | 命令行参数提供的可选覆盖配置 |
| `ProjectConfig` | 项目级别的配置（主要是信任级别） |
| `ToolsToml` | 工具功能开关的配置部分 |
| `SandboxPolicyResolution` | 沙箱策略解析结果 |

## 主要常量

| 常量名称 | 值 | 说明 |
|---------|---|------|
| `OPENAI_DEFAULT_MODEL` | `"gpt-5-codex"` (非 Windows) / `"gpt-5"` (Windows) | 默认 OpenAI 模型 |
| `OPENAI_DEFAULT_REVIEW_MODEL` | `"gpt-5-codex"` | 默认代码审查模型 |
| `GPT_5_CODEX_MEDIUM_MODEL` | `"gpt-5-codex"` | GPT-5 Codex 中等模型 |
| `PROJECT_DOC_MAX_BYTES` | `32 * 1024` | 项目文档最大字节数（32 KiB） |
| `CONFIG_TOML_FILE` | `"config.toml"` | 配置文件名 |

## 主要函数/方法

### Config 加载和初始化

| 函数/方法 | 说明 |
|----------|------|
| `Config::load_with_cli_overrides` | 加载配置并应用 CLI 覆盖 |
| `Config::load_from_base_config_with_overrides` | 从基础配置和覆盖选项构建最终配置 |
| `load_config_as_toml_with_cli_overrides` | 加载配置为 TOML 格式并应用 CLI 覆盖 |
| `load_resolved_config` | 加载并解析分层配置 |

### 配置合并和应用

| 函数/方法 | 说明 |
|----------|------|
| `apply_overlays` | 应用 CLI 覆盖和托管配置层 |
| `apply_toml_override` | 应用单个点分隔路径的 TOML 覆盖 |

### MCP 服务器管理

| 函数/方法 | 说明 |
|----------|------|
| `load_global_mcp_servers` | 加载全局 MCP 服务器配置 |
| `ensure_no_inline_bearer_tokens` | 确保没有不安全的内联 bearer token |

### 项目信任管理

| 函数/方法 | 说明 |
|----------|------|
| `set_project_trusted` | 将项目标记为受信任 |
| `set_project_trusted_inner` | 内部实现，更新 TOML 文档中的项目信任设置 |

### ConfigToml 方法

| 方法 | 说明 |
|-----|------|
| `derive_sandbox_policy` | 从配置派生有效的沙箱策略 |
| `get_active_project` | 获取当前工作目录对应的活动项目配置 |
| `get_config_profile` | 获取指定名称的配置 Profile |

### ProjectConfig 方法

| 方法 | 说明 |
|-----|------|
| `is_trusted` | 检查项目是否被标记为受信任 |

## 配置层次结构

配置按以下优先级合并（从高到低）：

1. **命令行覆盖** (`ConfigOverrides`)
2. **托管首选项** (macOS managed device profiles)
3. **托管配置** (`managed_config.toml`)
4. **活动 Profile** (`profiles.<name>`)
5. **用户配置** (`config.toml`)
6. **默认值** (代码中定义的默认值)

## 与其他模块的关系

- **使用 `config/types.rs`**：所有配置数据类型
- **使用 `config/profile.rs`**：Profile 配置结构
- **使用 `config/edit.rs`**：配置持久化功能
- **使用 `config_loader/mod.rs`**：配置文件加载逻辑
- **与 `auth` 模块交互**：认证凭据存储模式
- **与 `features` 模块交互**：功能开关管理
- **与 `model_provider_info` 交互**：模型提供商信息
- **与协议层交互**：转换为 app-server 协议类型

## 主要配置项

### 模型配置
- 模型选择、提供商、上下文窗口、输出限制
- 推理努力程度、推理摘要、详细程度

### 安全配置
- 审批策略（命令执行前是否需要确认）
- 沙箱模式（只读、工作空间写入、完全访问）
- Shell 环境策略（环境变量继承和过滤）

### 工具配置
- MCP 服务器定义和配置
- 内置工具开关（web 搜索、图片查看等）
- 实验性功能开关

### 项目配置
- 项目文档（AGENTS.md）设置
- 项目信任级别
- 工作目录解析

### UI 配置
- 历史记录持久化
- 通知设置
- 文件打开器集成
- 代理推理显示选项

## 设计特点

1. **分层配置**：支持多层配置源的合并
2. **类型安全**：强类型配置结构，编译时检查
3. **灵活覆盖**：支持命令行、Profile、托管配置等多种覆盖机制
4. **默认值策略**：为所有配置项提供合理的默认值
5. **平台适配**：根据操作系统调整默认配置（如 Windows 沙箱支持）
6. **Git 集成**：支持基于 Git 仓库的项目信任继承
