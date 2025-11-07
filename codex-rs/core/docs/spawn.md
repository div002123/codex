# spawn.rs

## 文件作用

提供进程生成功能，确保子进程继承正确的环境变量、工作目录和标准输入输出配置。

该模块是所有子进程创建的统一入口，处理跨平台的进程生成细节。

## 环境变量常量

### CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR
```rust
pub const CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR: &str = "CODEX_SANDBOX_NETWORK_DISABLED";
```

当以下两个条件都为真时，此变量将被设置为非空值：
1. 进程由 Codex 作为 shell 工具调用的一部分生成
2. `SandboxPolicy.has_full_network_access()` 返回 `false`

**用途**：允许子进程知道网络访问已被限制。

### CODEX_SANDBOX_ENV_VAR
```rust
pub const CODEX_SANDBOX_ENV_VAR: &str = "CODEX_SANDBOX";
```

当进程在沙箱下生成时设置。

**当前值**：
- macOS：`"seatbelt"`
- 其他平台：可能有不同的值

## 枚举类型

### StdioPolicy
```rust
pub enum StdioPolicy {
    RedirectForShellTool,
    Inherit,
}
```

标准输入输出策略。

- `RedirectForShellTool`：为 shell 工具重定向（stdin=null, stdout/stderr=piped）
- `Inherit`：继承父进程的 stdio

## 主要函数

### spawn_child_async
```rust
pub(crate) async fn spawn_child_async(
    program: PathBuf,
    args: Vec<String>,
    arg0: Option<&str>,
    cwd: PathBuf,
    sandbox_policy: &SandboxPolicy,
    stdio_policy: StdioPolicy,
    env: HashMap<String, String>,
) -> std::io::Result<Child>
```

生成子进程的主函数。

**参数**：
- `program`：要执行的程序路径
- `args`：命令行参数
- `arg0`：自定义 argv[0]（仅 Unix）
- `cwd`：工作目录
- `sandbox_policy`：沙箱策略（用于设置环境变量）
- `stdio_policy`：标准输入输出策略
- `env`：环境变量映射

**功能**：
1. 创建 `tokio::process::Command`
2. 设置 argv[0]（Unix）
3. 设置命令行参数
4. 设置工作目录
5. 清空环境变量并设置新的环境变量
6. 如果网络被限制，设置 `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR`
7. 配置父进程死亡信号（Linux）
8. 配置标准输入输出
9. 启用 `kill_on_drop`
10. 生成子进程

## 平台特定功能

### Unix (arg0 设置)
```rust
#[cfg(unix)]
cmd.arg0(arg0.map_or_else(|| program.to_string_lossy().to_string(), String::from));
```
允许自定义 argv[0]，用于某些需要特定程序名的场景。

### Linux (父进程死亡信号)
```rust
#[cfg(target_os = "linux")]
unsafe {
    let parent_pid = libc::getpid();
    cmd.pre_exec(move || {
        if libc::prctl(libc::PR_SET_PDEATHSIG, libc::SIGTERM) == -1 {
            return Err(std::io::Error::last_os_error());
        }
        if libc::getppid() != parent_pid {
            libc::raise(libc::SIGTERM);
        }
        Ok(())
    });
}
```

使用 `prctl(PR_SET_PDEATHSIG, SIGTERM)` 确保：
- 当父进程（Codex）死亡时，子进程收到 SIGTERM 信号
- 避免孤儿进程继续运行
- 处理竞态条件：如果父进程已死亡，立即自杀

## 标准输入输出配置

### RedirectForShellTool
```rust
cmd.stdin(Stdio::null());
cmd.stdout(Stdio::piped()).stderr(Stdio::piped());
```

**stdin**：设置为 `null` 以防止命令挂起等待输入。

**原因**：某些命令（如 ripgrep）有启发式检测，可能尝试从 stdin 读取，导致无限等待。

**stdout/stderr**：重定向到管道，供父进程读取。

### Inherit
```rust
cmd.stdin(Stdio::inherit())
   .stdout(Stdio::inherit())
   .stderr(Stdio::inherit());
```

继承父进程的标准输入输出，适用于交互式场景。

## 环境变量处理

```rust
cmd.env_clear();
cmd.envs(env);
```

1. **清空所有环境变量**：防止意外继承敏感变量
2. **设置指定的环境变量**：只设置 `env` 映射中的变量

**沙箱相关变量**：
```rust
if !sandbox_policy.has_full_network_access() {
    cmd.env(CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR, "1");
}
```

## 进程生命周期管理

### kill_on_drop
```rust
cmd.kill_on_drop(true).spawn()
```

当 `Child` 对象被丢弃时自动终止子进程。

**好处**：
- 防止进程泄漏
- 确保异常退出时清理子进程

## 使用示例

### 示例 1：基本进程生成
```rust
let child = spawn_child_async(
    PathBuf::from("/bin/ls"),
    vec!["-la".to_string()],
    None,
    PathBuf::from("/tmp"),
    &SandboxPolicy::DangerFullAccess,
    StdioPolicy::RedirectForShellTool,
    HashMap::from([("PATH".to_string(), "/usr/bin".to_string())]),
).await?;
```

### 示例 2：自定义 argv[0]
```rust
let child = spawn_child_async(
    PathBuf::from("/usr/bin/python3"),
    vec!["script.py".to_string()],
    Some("custom-name"),  // argv[0] 将是 "custom-name"
    cwd,
    &SandboxPolicy::ReadOnly,
    StdioPolicy::RedirectForShellTool,
    env,
).await?;
```

### 示例 3：继承 stdio
```rust
let child = spawn_child_async(
    program,
    args,
    None,
    cwd,
    &SandboxPolicy::DangerFullAccess,
    StdioPolicy::Inherit,  // 子进程将继承父进程的 stdio
    env,
).await?;
```

## 安全考虑

1. **环境变量清理**：始终调用 `env_clear()` 防止泄露敏感信息
2. **父进程死亡处理**：Linux 上使用 `prctl` 确保子进程不会成为孤儿进程
3. **沙箱标记**：通过环境变量通知子进程沙箱状态
4. **stdin 保护**：`RedirectForShellTool` 模式下 stdin 设为 null 防止挂起

## 与其他模块的关系

- **exec.rs**：主要使用者，用于执行所有命令
- **exec_env.rs**：提供 `env` 参数的环境变量映射
- **protocol.rs**：使用 `SandboxPolicy` 类型
- **unified_exec**：通过 PTY 库间接使用（PTY 库内部可能使用类似逻辑）
- **sandboxing**：沙箱模块使用此函数生成沙箱化的子进程
- **tokio**：依赖 tokio 的异步进程 API
