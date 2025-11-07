# shell.rs

## 文件作用

检测和配置用户的默认 shell，支持 Zsh、Bash 和 PowerShell。

该模块提供跨平台的 shell 检测功能，并为每种 shell 提供相应的配置路径和执行参数。

## 主要结构体

### ZshShell
```rust
pub struct ZshShell {
    pub(crate) shell_path: String,
    pub(crate) zshrc_path: String,
}
```

Zsh shell 配置。

- `shell_path`：Zsh 可执行文件路径（如 `/bin/zsh`）
- `zshrc_path`：Zshrc 配置文件路径（如 `~/.zshrc`）

### BashShell
```rust
pub struct BashShell {
    pub(crate) shell_path: String,
    pub(crate) bashrc_path: String,
}
```

Bash shell 配置。

- `shell_path`：Bash 可执行文件路径（如 `/bin/bash`）
- `bashrc_path`：Bashrc 配置文件路径（如 `~/.bashrc`）

### PowerShellConfig
```rust
pub struct PowerShellConfig {
    pub(crate) exe: String,
    pub(crate) bash_exe_fallback: Option<PathBuf>,
}
```

PowerShell 配置。

- `exe`：可执行文件名或路径（如 `pwsh` 或 `powershell.exe`）
- `bash_exe_fallback`：Bash 回退路径（当模型生成 bash 命令时使用）

### Shell
```rust
pub enum Shell {
    Zsh(ZshShell),
    Bash(BashShell),
    PowerShell(PowerShellConfig),
    Unknown,
}
```

Shell 枚举类型。

**方法**：
```rust
pub fn name(&self) -> Option<String>
```
返回 shell 的名称（如 `"zsh"`, `"bash"`, `"pwsh"`）。

## 主要函数

### default_user_shell
```rust
pub async fn default_user_shell() -> Shell
```

检测用户的默认 shell（平台特定实现）。

#### Unix 实现（Linux、macOS）
```rust
fn detect_default_user_shell() -> Shell
```

使用 `getpwuid` 系统调用获取用户信息：
1. 获取当前用户 UID
2. 查询用户数据库获取 shell 路径和 home 目录
3. 根据 shell 路径后缀判断类型：
   - 以 `/zsh` 结尾 → `Shell::Zsh`
   - 以 `/bash` 结尾 → `Shell::Bash`
   - 其他 → `Shell::Unknown`

#### Windows 实现
检测逻辑：
1. 尝试运行 `pwsh` 检查 PowerShell 7+ 是否可用
2. 检查 `bash.exe` 是否可用（如 WSL 或 Git Bash）
3. 返回结果：
   - 如果 `pwsh` 可用：`PowerShell { exe: "pwsh.exe", bash_exe_fallback }`
   - 否则：`PowerShell { exe: "powershell.exe", bash_exe_fallback }`

## 平台特定实现

### Unix (Linux/macOS)
使用 `libc` 的 `getpwuid` 和 `getuid` 系统调用：
```rust
unsafe {
    let uid = getuid();
    let pw = getpwuid(uid);
    let shell_path = CStr::from_ptr((*pw).pw_shell).to_string_lossy();
    let home_path = CStr::from_ptr((*pw).pw_dir).to_string_lossy();
}
```

### Windows
使用 `tokio::process::Command` 执行命令检测：
```rust
// 检查 pwsh
Command::new("pwsh")
    .arg("-NoLogo")
    .arg("-NoProfile")
    .arg("-Command")
    .arg("$PSVersionTable.PSVersion.Major")
    .output()
    .await

// 检查 bash.exe
Command::new("bash.exe")
    .arg("--version")
    .output()
    .await
```

### 其他平台
返回 `Shell::Unknown`。

## 测试功能

### Unix 测试
- `test_current_shell_detects_zsh`：验证 Zsh 检测
- `test_run_with_profile_bash_escaping_and_execution`：测试 Bash profile 加载和命令执行

### macOS 测试
- `test_run_with_profile_escaping_and_execution`：测试 Zsh profile 加载和复杂命令执行

### Windows 测试
- `test_format_default_shell_invocation_powershell`：测试 PowerShell 命令格式化

## 使用示例

### 示例 1：检测默认 shell
```rust
let shell = default_user_shell().await;
match shell {
    Shell::Zsh(config) => {
        println!("Zsh: {}", config.shell_path);
        println!("Config: {}", config.zshrc_path);
    }
    Shell::Bash(config) => {
        println!("Bash: {}", config.shell_path);
        println!("Config: {}", config.bashrc_path);
    }
    Shell::PowerShell(config) => {
        println!("PowerShell: {}", config.exe);
    }
    Shell::Unknown => {
        println!("Unknown shell");
    }
}
```

### 示例 2：获取 shell 名称
```rust
let shell = default_user_shell().await;
if let Some(name) = shell.name() {
    println!("Shell name: {}", name);
}
```

## 与其他模块的关系

- **exec.rs**：使用 shell 配置执行命令
- **bash.rs**：配合使用，解析 bash/zsh 命令
- **exec_env.rs**：shell 环境变量配置
- **spawn.rs**：使用 shell 路径生成子进程
- **protocol.rs**：shell 配置可能影响执行策略
- **config**：shell 配置可能来自用户配置文件

## 注意事项

1. **安全性**：Unix 实现使用 `unsafe` 代码调用 C 库函数
2. **异步**：`default_user_shell` 是异步函数（Windows 需要执行子进程）
3. **回退**：Windows 上 PowerShell 7+ 优先，回退到 Windows PowerShell
4. **路径处理**：配置文件路径基于 home 目录构建
5. **WSL 支持**：Windows 上检测 bash.exe 以支持 WSL 环境
