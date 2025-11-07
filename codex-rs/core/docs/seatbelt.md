# seatbelt.rs - macOS 沙箱接口

## 文件作用

该文件提供了 macOS Seatbelt 沙箱的接口实现，负责生成和执行 macOS 系统级沙箱策略。

## 平台限制

- **仅 macOS**: 整个文件使用 `#![cfg(target_os = "macos")]` 保护
- **依赖系统工具**: 需要 `/usr/bin/sandbox-exec`

## 主要常量

### MACOS_PATH_TO_SEATBELT_EXECUTABLE
- **值**: `"/usr/bin/sandbox-exec"`
- **作用**: Seatbelt 可执行文件的固定路径
- **安全考虑**: 仅使用 `/usr/bin` 中的版本，防止 PATH 注入攻击

### MACOS_SEATBELT_BASE_POLICY
- **来源**: `include_str!("seatbelt_base_policy.sbpl")`
- **作用**: 基础沙箱策略文本
- **语言**: Seatbelt Policy Language (SBPL)

### MACOS_SEATBELT_NETWORK_POLICY
- **来源**: `include_str!("seatbelt_network_policy.sbpl")`
- **作用**: 网络访问策略文本
- **启用条件**: 当 `sandbox_policy.has_full_network_access()` 为 true

## 主要函数

### spawn_command_under_seatbelt
- **签名**: `pub async fn spawn_command_under_seatbelt(...) -> std::io::Result<Child>`
- **作用**: 在 Seatbelt 沙箱中生成子进程
- **参数**:
  - `command: Vec<String>` - 待执行的命令
  - `command_cwd: PathBuf` - 命令工作目录
  - `sandbox_policy: &SandboxPolicy` - 沙箱策略
  - `sandbox_policy_cwd: &Path` - 策略基准目录
  - `stdio_policy: StdioPolicy` - 标准输入输出策略
  - `env: HashMap<String, String>` - 环境变量
- **特殊处理**: 设置 `CODEX_SANDBOX=seatbelt` 环境变量

### create_seatbelt_command_args
- **签名**: `pub(crate) fn create_seatbelt_command_args(...) -> Vec<String>`
- **作用**: 构建 `sandbox-exec` 命令行参数
- **返回**: 包含策略和命令的完整参数列表

## Seatbelt 策略构建

### 文件写入策略

#### 完全磁盘写入
```scheme
(allow file-write* (regex #"^/"))
```
- 条件: `sandbox_policy.has_full_disk_write_access()`

#### 受限写入
- 基于 `writable_roots` 列表
- 使用 `(subpath (param "WRITABLE_ROOT_N"))` 指定可写路径
- 支持只读子路径排除：
  ```scheme
  (require-all
    (subpath (param "WRITABLE_ROOT_0"))
    (require-not (subpath (param "WRITABLE_ROOT_0_RO_0"))))
  ```

### 文件读取策略

#### 完全磁盘读取
```scheme
(allow file-read*)
```
- 条件: `sandbox_policy.has_full_disk_read_access()`
- 默认启用（大多数策略）

### 网络策略
- 条件性包含 `MACOS_SEATBELT_NETWORK_POLICY`
- 基于 `sandbox_policy.has_full_network_access()`

### 完整策略组装
```scheme
{BASE_POLICY}
{file_read_policy}
{file_write_policy}
{network_policy}
```

## 路径处理

### 路径规范化
- 使用 `canonicalize()` 转换为规范路径
- **原因**: 避免 `/var` vs `/private/var` 等符号链接不匹配
- **回退**: 规范化失败时使用原始路径

### 参数化路径
- 使用 `-D` 标志定义路径参数
- 格式: `-DWRITABLE_ROOT_0=/canonical/path`
- 在策略中引用: `(param "WRITABLE_ROOT_0")`

### .git 目录特殊处理
- 自动排除可写根目录下的 `.git` 子目录
- 使用 `read_only_subpaths` 机制
- 防止意外修改版本控制数据

## macOS 系统集成

### confstr 函数族
- **confstr()**: 包装 `libc::confstr()` 返回字符串
- **confstr_path()**: 包装并转换为 PathBuf
- **用途**: 获取系统目录路径

### macos_dir_params
- **作用**: 获取 macOS 特定的目录参数
- **当前实现**: 获取 `_CS_DARWIN_USER_CACHE_DIR`
- **返回**: `Vec<(String, PathBuf)>` 参数对

## 命令格式

### 完整命令示例
```bash
/usr/bin/sandbox-exec \
  -p '(version 1) (allow default) ...' \
  -DWRITABLE_ROOT_0=/Users/me/project \
  -DWRITABLE_ROOT_0_RO_0=/Users/me/project/.git \
  -DWRITABLE_ROOT_1=/tmp \
  -DDARWIN_USER_CACHE_DIR=/Users/me/Library/Caches \
  -- \
  bash -c "echo hello"
```

### 参数说明
- `-p` - 内联策略文本
- `-D<NAME>=<VALUE>` - 定义策略参数
- `--` - 分隔沙箱参数和实际命令
- 后续 - 实际执行的命令

## 安全特性

### 路径安全
1. **固定可执行文件路径**: 不依赖 PATH 环境变量
2. **路径规范化**: 避免符号链接绕过
3. **只读保护**: 自动保护 `.git` 目录

### 策略安全
1. **最小权限**: 默认拒绝所有操作
2. **显式允许**: 明确列出允许的操作
3. **路径参数化**: 使用变量而非硬编码路径

## 测试支持

### 测试辅助函数
- `populate_tmpdir()` - 创建测试目录结构
- 支持有/无 `.git` 目录的场景
- 验证 `.git` 自动排除逻辑

### 测试案例
1. 带 `.git` 子目录的可写根
2. cwd 作为 Git 仓库的情况
3. 参数格式正确性验证

## 与其他模块的关系

- **依赖 `spawn` 模块**: 使用 `spawn_child_async()` 创建进程
- **依赖 `protocol::SandboxPolicy`**: 定义沙箱策略
- **被 `sandboxing::mod.rs` 调用**: 提供 macOS 平台的沙箱实现
- **与 `landlock.rs` 并列**: 分别处理不同平台
- **使用系统工具**: 依赖 macOS 系统的 `sandbox-exec`
- **策略文件**: 包含 `.sbpl` 策略模板文件
