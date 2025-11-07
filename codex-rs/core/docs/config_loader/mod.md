# mod.rs

## 文件作用

该文件实现了配置加载器，负责从多个来源加载配置文件并按正确的优先级合并它们。支持三层配置：用户配置、托管配置和托管首选项（仅 macOS）。

配置加载器的核心职责是读取 TOML 配置文件、处理缺失文件的情况，以及实现配置层的合并逻辑。

## 配置层次架构

```
+-------------------------+
| Managed preferences (*) |  ← 最高优先级（仅 macOS）
+-------------------------+
           ↑
           |
+-------------------------+
|  managed_config.toml   |  ← 中等优先级
+-------------------------+
           ↑
           |
+-------------------------+
|    config.toml (base)   |  ← 基础配置
+-------------------------+
```

(*) 仅在 macOS 上通过托管设备配置文件可用

## 主要结构体

| 结构体名称 | 说明 |
|----------|------|
| `LoadedConfigLayers` | 包含所有已加载配置层的结构体 |
| `LoaderOverrides` | 配置加载器的可选覆盖，主要用于测试 |

## 主要函数

| 函数 | 说明 |
|-----|------|
| `load_config_as_toml` | 加载并合并所有配置层，返回最终的 TOML 值 |
| `load_config_as_toml_with_overrides` | 带覆盖选项的配置加载（主要用于测试） |
| `load_config_layers_with_overrides` | 加载所有配置层但不合并（返回分离的层） |
| `load_config_layers_internal` | 内部实现，加载所有配置层 |
| `read_config_from_path` | 从指定路径读取并解析单个配置文件 |
| `merge_toml_values` | 递归合并两个 TOML 值，覆盖层优先 |
| `managed_config_default_path` | 获取托管配置的默认路径 |
| `apply_managed_layers` | 将托管配置层应用到基础配置 |

## 配置文件路径

### Unix/Linux/macOS
- **用户配置**：`~/.codex/config.toml`
- **托管配置**：`/etc/codex/managed_config.toml`
- **托管首选项** (仅 macOS)：通过 Core Foundation API 从系统首选项读取

### Windows
- **用户配置**：`%USERPROFILE%\.codex\config.toml`
- **托管配置**：`%USERPROFILE%\.codex\managed_config.toml`

## 主要常量

| 常量 | 值 | 说明 |
|-----|---|------|
| `CODEX_MANAGED_CONFIG_SYSTEM_PATH` | `/etc/codex/managed_config.toml` | Unix 系统上的托管配置路径 |

## 错误处理

- **文件不存在**：作为 info 级别日志记录（用户配置）或 debug 级别（托管配置），不报错
- **解析失败**：作为 error 级别日志并返回错误
- **读取失败**：作为 error 级别日志并返回错误

## 与其他模块的关系

- **被 `config/mod.rs` 使用**：提供配置文件加载功能
- **使用 `config_loader/macos.rs`**：在 macOS 上加载托管首选项
- **使用 `tokio::fs`**：异步文件读取
- **使用 `toml` 库**：TOML 解析

## 设计特点

1. **异步 IO**：所有文件读取操作都是异步的，避免阻塞
2. **优雅降级**：缺失的配置文件不会导致错误，返回空配置
3. **递归合并**：支持深度合并嵌套的 TOML 表
4. **平台适配**：根据操作系统选择正确的配置路径和加载策略
5. **测试友好**：通过 `LoaderOverrides` 支持测试时注入自定义配置路径
6. **日志完善**：对配置加载过程提供详细的日志记录

## 合并逻辑

`merge_toml_values` 函数实现了配置合并的核心逻辑：

- 如果两个值都是表（Table），递归合并键值对
- 覆盖层中的键会覆盖基础层中的同名键
- 非表类型的值直接替换
- 保持覆盖层优先的原则
