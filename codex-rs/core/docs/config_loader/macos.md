# macos.rs

## 文件作用

该文件实现了 macOS 特定的托管配置加载功能。它通过 Core Foundation API 读取由 MDM（移动设备管理）或企业配置文件设置的托管首选项。

在 macOS 上，管理员可以通过配置文件（Configuration Profiles）为企业环境下的 Codex 部署集中管理配置。该模块负责读取这些托管首选项并将其集成到配置系统中。

## 主要函数

### macOS 平台（`target_os = "macos"`）

| 函数 | 说明 |
|-----|------|
| `load_managed_admin_config_layer` | 异步加载托管管理员配置层，支持覆盖（用于测试） |
| `load_managed_admin_config` | 同步加载托管管理员配置（在后台线程中执行） |
| `parse_managed_preferences_base64` | 解析 Base64 编码的托管首选项 TOML |

### 非 macOS 平台

| 函数 | 说明 |
|-----|------|
| `load_managed_admin_config_layer` | 空实现，始终返回 `None` |

## 托管首选项参数

- **Application ID**：`com.openai.codex`
- **配置键名**：`config_toml_base64`
- **值格式**：Base64 编码的 TOML 配置内容

## 工作流程

1. **读取系统首选项**：通过 `CFPreferencesCopyAppValue` 读取托管首选项
2. **检查是否存在**：如果首选项不存在，返回 `None`
3. **Base64 解码**：将读取的 Base64 字符串解码为字节
4. **UTF-8 验证**：确保解码后的内容是有效的 UTF-8 文本
5. **TOML 解析**：将文本解析为 TOML 值
6. **表验证**：确保根元素是 TOML 表（而非其他类型）

## Core Foundation API 使用

```rust
#[link(name = "CoreFoundation", kind = "framework")]
unsafe extern "C" {
    fn CFPreferencesCopyAppValue(
        key: CFStringRef,
        application_id: CFStringRef,
    ) -> *mut c_void;
}
```

- 使用 `core-foundation` crate 的 Rust 绑定
- 安全封装了底层的 C API 调用

## 错误处理

所有错误都会被记录并转换为 `io::Error`：

| 错误类型 | 处理方式 |
|---------|---------|
| 首选项不存在 | Debug 日志，返回 `None` |
| Base64 解码失败 | Error 日志，返回 `InvalidData` 错误 |
| UTF-8 验证失败 | Error 日志，返回 `InvalidData` 错误 |
| TOML 解析失败 | Error 日志，返回 `InvalidData` 错误 |
| 根不是表类型 | Error 日志，返回 `InvalidData` 错误 |
| 异步任务失败 | Error 日志，返回通用错误 |

## 与其他模块的关系

- **被 `config_loader/mod.rs` 使用**：作为配置加载流程的一部分
- **使用 Core Foundation**：通过 `core-foundation` crate 访问 macOS API
- **使用 `base64`**：解码 Base64 编码的配置
- **使用 `tokio::task`**：在后台线程中执行阻塞的系统调用

## 设计特点

1. **平台条件编译**：使用 `#[cfg(target_os = "macos")]` 实现平台特定功能
2. **异步封装**：将阻塞的系统 API 调用封装在 `spawn_blocking` 中
3. **测试支持**：支持通过参数注入 Base64 编码的配置用于测试
4. **类型安全**：使用 Core Foundation 的 Rust 绑定确保内存安全
5. **错误详细**：为每个错误情况提供详细的日志信息

## 企业部署场景

管理员可以通过以下方式部署托管配置：

1. 创建包含配置的 `.mobileconfig` 文件
2. 在配置文件中设置 `com.openai.codex` 应用 ID
3. 添加 `config_toml_base64` 键，值为 Base64 编码的 TOML 配置
4. 通过 MDM 系统或手动安装配置文件
5. Codex 启动时自动读取并应用这些设置

这种机制允许 IT 部门集中管理 Codex 的配置，例如：
- 强制使用特定的 API 端点
- 限制可用的模型
- 配置企业代理设置
- 设置统一的沙箱策略
