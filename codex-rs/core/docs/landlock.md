# landlock.rs - Linux 沙箱接口

## 文件作用

该文件提供了 Linux 平台沙箱（Landlock + seccomp）的接口实现，负责将命令包装在 `codex-linux-sandbox` 辅助进程中执行。

## 主要函数

### spawn_command_under_linux_sandbox
- **签名**: `pub async fn spawn_command_under_linux_sandbox<P>(...) -> std::io::Result<Child>`
- **作用**: 在 Linux 沙箱中生成子进程
- **参数**:
  - `codex_linux_sandbox_exe: P` - 沙箱可执行文件路径
  - `command: Vec<String>` - 待执行的命令
  - `command_cwd: PathBuf` - 命令的工作目录
  - `sandbox_policy: &SandboxPolicy` - 沙箱策略
  - `sandbox_policy_cwd: &Path` - 策略的基准目录
  - `stdio_policy: StdioPolicy` - 标准输入输出策略
  - `env: HashMap<String, String>` - 环境变量
- **返回**: 生成的子进程对象
- **实现**: 调用 `spawn_child_async()`

### create_linux_sandbox_command_args
- **签名**: `pub(crate) fn create_linux_sandbox_command_args(...) -> Vec<String>`
- **作用**: 构建 `codex-linux-sandbox` 的命令行参数
- **生成格式**:
  ```bash
  codex-linux-sandbox \
    --sandbox-policy-cwd <cwd> \
    --sandbox-policy <json> \
    -- \
    <actual_command> <args...>
  ```

## Linux 沙箱架构

### 设计特点
1. **外部辅助进程**: 使用独立的 `codex-linux-sandbox` 可执行文件
2. **JSON 策略传递**: 将 `SandboxPolicy` 序列化为 JSON 传递
3. **双重隔离**:
   - **Landlock**: 文件系统访问控制
   - **seccomp**: 系统调用过滤

### 与 macOS Seatbelt 的对比
- **macOS**: 直接嵌入策略文本到 `sandbox-exec` 命令
- **Linux**: 使用 CLI 标志传递策略，辅助进程解析并应用

## 命令参数格式

### 完整命令示例
```bash
/path/to/codex-linux-sandbox \
  --sandbox-policy-cwd /home/user/project \
  --sandbox-policy '{"WorkspaceWrite":{"writable_roots":[],"network_access":false}}' \
  -- \
  bash -c "ls -la"
```

### 参数说明
- `--sandbox-policy-cwd` - 策略基准目录（通常是项目根目录）
- `--sandbox-policy` - 序列化的 SandboxPolicy JSON
- `--` - 分隔符，防止命令参数被误解析为沙箱选项
- 后续参数 - 实际要执行的命令及其参数

## 错误处理

### 潜在错误
1. **可执行文件不存在**: 找不到 `codex-linux-sandbox`
2. **权限不足**: 无法执行沙箱辅助进程
3. **策略无效**: JSON 序列化失败（使用 `expect`，不应失败）
4. **路径非 UTF-8**: cwd 包含无效字符（使用 `expect`）

### 安全假设
- 使用 `expect()` 的地方假设：
  - `SandboxPolicy` 总是可以序列化为 JSON
  - `cwd` 路径总是有效的 UTF-8 字符串

## 进程管理

### arg0 覆盖
- 设置为 `"codex-linux-sandbox"`
- 用于进程列表中的显示名称
- 便于识别和调试

### 异步执行
- 使用 `spawn_child_async()` 创建进程
- 返回 `tokio::process::Child`
- 支持异步等待和信号处理

## 与其他模块的关系

- **依赖 `spawn` 模块**: 使用 `spawn_child_async()` 创建进程
- **依赖 `protocol::SandboxPolicy`**: 定义沙箱策略结构
- **被 `sandboxing::mod.rs` 调用**: 提供 Linux 平台的沙箱实现
- **与 `seatbelt.rs` 并列**: 分别处理 macOS 和 Linux 平台
- **配合外部二进制**: 需要 `codex-linux-sandbox` 可执行文件
