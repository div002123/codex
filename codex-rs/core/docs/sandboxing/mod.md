# sandboxing/mod.rs - 沙箱管理模块

## 文件作用

该文件是沙箱系统的核心模块，负责构建平台相关的沙箱包装器，并将可移植的 `CommandSpec` 转换为可执行的 `ExecEnv`。

## 主要结构体

### CommandSpec
- **作用**: 命令规范，描述待执行的命令
- **字段**:
  - `program: String` - 程序路径
  - `args: Vec<String>` - 命令参数
  - `cwd: PathBuf` - 工作目录
  - `env: HashMap<String, String>` - 环境变量
  - `timeout_ms: Option<u64>` - 超时时间（毫秒）
  - `with_escalated_permissions: Option<bool>` - 是否需要提升权限
  - `justification: Option<String>` - 执行理由

### ExecEnv
- **作用**: 执行环境，包含完整的沙箱化命令
- **字段**:
  - `command: Vec<String>` - 完整命令（包含沙箱包装器）
  - `cwd: PathBuf` - 工作目录
  - `env: HashMap<String, String>` - 环境变量
  - `timeout_ms: Option<u64>` - 超时时间
  - `sandbox: SandboxType` - 沙箱类型
  - `with_escalated_permissions: Option<bool>` - 权限提升标志
  - `justification: Option<String>` - 执行理由
  - `arg0: Option<String>` - 进程名称覆盖

### SandboxManager
- **作用**: 沙箱管理器，处理沙箱选择和命令转换
- **方法**:
  - `select_initial()` - 选择初始沙箱类型
  - `transform()` - 将 CommandSpec 转换为 ExecEnv
  - `denied()` - 检查命令是否被沙箱拒绝

### SandboxPreference
- **作用**: 沙箱偏好设置
- **变体**:
  - `Auto` - 自动选择
  - `Require` - 要求使用沙箱
  - `Forbid` - 禁止沙箱

## 主要枚举

### SandboxTransformError
- **作用**: 沙箱转换错误类型
- **变体**:
  - `MissingLinuxSandboxExecutable` - 缺少 Linux 沙箱可执行文件
  - `SeatbeltUnavailable` - Seatbelt 仅在 macOS 上可用

## 核心功能

### 沙箱选择逻辑 (select_initial)
- **基于策略**:
  - `DangerFullAccess` → 无沙箱
  - 其他策略 → 平台沙箱（如果可用）
- **基于偏好**:
  - `Forbid` → 无沙箱
  - `Require` → 强制使用平台沙箱
  - `Auto` → 根据策略决定

### 命令转换逻辑 (transform)

#### 无沙箱模式
- 直接使用原始命令
- 设置网络禁用环境变量（如需要）

#### macOS Seatbelt
- 调用 `/usr/bin/sandbox-exec`
- 使用 `create_seatbelt_command_args()` 生成参数
- 设置 `CODEX_SANDBOX=seatbelt` 环境变量

#### Linux Seccomp
- 调用 `codex-linux-sandbox` 可执行文件
- 使用 `create_linux_sandbox_command_args()` 生成参数
- 设置 `arg0` 为 "codex-linux-sandbox"

#### Windows Restricted Token
- 保持命令不变
- 在执行时通过 `codex-windows-sandbox` 库处理

## 环境变量

- `CODEX_SANDBOX_NETWORK_DISABLED` - 指示网络已被禁用
- `CODEX_SANDBOX` (macOS) - 指示沙箱类型 ("seatbelt")

## 辅助函数

### execute_env
- **作用**: 异步执行 ExecEnv
- **签名**: `async fn execute_env(env: &ExecEnv, policy: &SandboxPolicy, stdout_stream: Option<StdoutStream>) -> Result<ExecToolCallOutput>`
- **功能**: 委托给 `execute_exec_env()`

## 与其他模块的关系

- **依赖 `landlock.rs`**: Linux 沙箱命令构建
- **依赖 `seatbelt.rs`**: macOS 沙箱命令构建
- **依赖 `exec` 模块**: 执行沙箱化的命令
- **依赖 `protocol::SandboxPolicy`**: 策略定义
- **被 `tools` 模块使用**: 工具执行的沙箱化
- **子模块 `assessment`**: 沙箱命令风险评估

## 子模块

- `assessment` - 沙箱命令评估，使用 AI 模型分析命令风险
