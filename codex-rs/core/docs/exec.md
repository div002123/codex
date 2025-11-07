# exec.rs

## 文件作用

提供命令执行的核心功能，包括进程生成、输出捕获、超时处理、沙箱集成和沙箱拒绝检测。

这是 Codex 中所有命令执行的基础层，支持流式输出、超时控制和多种沙箱机制。

## 主要结构体

### ExecParams
```rust
pub struct ExecParams {
    pub command: Vec<String>,
    pub cwd: PathBuf,
    pub timeout_ms: Option<u64>,
    pub env: HashMap<String, String>,
    pub with_escalated_permissions: Option<bool>,
    pub justification: Option<String>,
    pub arg0: Option<String>,
}
```

命令执行参数。

**主要方法**：
- `timeout_duration() -> Duration`：返回超时时长，默认 10 秒

### SandboxType
```rust
pub enum SandboxType {
    None,
    MacosSeatbelt,      // 仅 macOS 可用
    LinuxSeccomp,       // 仅 Linux 可用
    WindowsRestrictedToken, // 仅 Windows 可用
}
```

沙箱类型枚举。

### StdoutStream
```rust
pub struct StdoutStream {
    pub sub_id: String,
    pub call_id: String,
    pub tx_event: Sender<Event>,
}
```

用于流式输出的事件发送器。

### StreamOutput
```rust
pub struct StreamOutput<T: Clone> {
    pub text: T,
    pub truncated_after_lines: Option<u32>,
}
```

流输出结构，可以是 `String` 或 `Vec<u8>`。

**主要方法**：
- `new(text: String) -> Self`：创建字符串输出
- `from_utf8_lossy() -> StreamOutput<String>`：将字节转换为字符串

### ExecToolCallOutput
```rust
pub struct ExecToolCallOutput {
    pub exit_code: i32,
    pub stdout: StreamOutput<String>,
    pub stderr: StreamOutput<String>,
    pub aggregated_output: StreamOutput<String>,
    pub duration: Duration,
    pub timed_out: bool,
}
```

执行输出结果。

## 主要函数

### process_exec_tool_call
```rust
pub async fn process_exec_tool_call(
    params: ExecParams,
    sandbox_type: SandboxType,
    sandbox_policy: &SandboxPolicy,
    sandbox_cwd: &Path,
    codex_linux_sandbox_exe: &Option<PathBuf>,
    stdout_stream: Option<StdoutStream>,
) -> Result<ExecToolCallOutput>
```

处理执行工具调用的主入口：
1. 解析命令和参数
2. 创建 `CommandSpec`
3. 使用 `SandboxManager` 转换为 `ExecEnv`
4. 调用 `execute_env` 执行

### execute_exec_env
```rust
pub(crate) async fn execute_exec_env(
    env: ExecEnv,
    sandbox_policy: &SandboxPolicy,
    stdout_stream: Option<StdoutStream>,
) -> Result<ExecToolCallOutput>
```

执行 `ExecEnv`：
1. 调用 `exec` 执行命令
2. 测量执行时间
3. 通过 `finalize_exec_result` 处理结果

### exec
```rust
async fn exec(
    params: ExecParams,
    sandbox: SandboxType,
    sandbox_policy: &SandboxPolicy,
    stdout_stream: Option<StdoutStream>,
) -> Result<RawExecToolCallOutput>
```

实际执行命令：
- Windows 沙箱：调用 `exec_windows_sandbox`
- 其他平台：使用 `spawn_child_async` 生成子进程

### consume_truncated_output
```rust
async fn consume_truncated_output(
    mut child: Child,
    timeout: Duration,
    stdout_stream: Option<StdoutStream>,
) -> Result<RawExecToolCallOutput>
```

消费子进程输出：
1. 生成任务读取 stdout 和 stderr
2. 使用 `read_capped` 收集输出
3. 通过聚合通道合并 stdout 和 stderr
4. 处理超时和 Ctrl+C 信号
5. 返回退出状态和输出

### read_capped
```rust
async fn read_capped<R: AsyncRead + Unpin + Send + 'static>(
    mut reader: R,
    stream: Option<StdoutStream>,
    is_stderr: bool,
    aggregate_tx: Option<Sender<Vec<u8>>>,
) -> io::Result<StreamOutput<Vec<u8>>>
```

读取流并：
1. 收集所有输出到缓冲区
2. 发送输出增量事件（限制为 10,000 个事件）
3. 将数据块发送到聚合通道

### is_likely_sandbox_denied
```rust
pub(crate) fn is_likely_sandbox_denied(
    sandbox_type: SandboxType,
    exec_output: &ExecToolCallOutput,
) -> bool
```

启发式检测沙箱拒绝：
1. 快速拒绝：非沙箱模式或退出码为 0
2. 检查输出中的沙箱拒绝关键词
3. 快速拒绝：特定的 shell 错误码（2, 126, 127）
4. 在 Linux Seccomp 下检查 SIGSYS 信号

**沙箱拒绝关键词**：
- "operation not permitted"
- "permission denied"
- "read-only file system"
- "seccomp"
- "sandbox"
- "landlock"
- "failed to write file"

### finalize_exec_result
```rust
fn finalize_exec_result(
    raw_output_result: std::result::Result<RawExecToolCallOutput, CodexErr>,
    sandbox_type: SandboxType,
    duration: Duration,
) -> Result<ExecToolCallOutput>
```

最终化执行结果：
1. 检查信号终止（Unix）
2. 转换退出码和超时状态
3. 将字节输出转换为 UTF-8
4. 如果超时，返回 `SandboxErr::Timeout`
5. 如果检测到沙箱拒绝，返回 `SandboxErr::Denied`

## Windows 特定功能

### exec_windows_sandbox
```rust
async fn exec_windows_sandbox(
    params: ExecParams,
    sandbox_policy: &SandboxPolicy,
) -> Result<RawExecToolCallOutput>
```

在 Windows 上使用 `codex_windows_sandbox` 执行命令。

## 常量

- `DEFAULT_TIMEOUT_MS`: 10,000 ms
- `SIGKILL_CODE`: 9
- `TIMEOUT_CODE`: 64
- `EXIT_CODE_SIGNAL_BASE`: 128
- `EXEC_TIMEOUT_EXIT_CODE`: 124
- `READ_CHUNK_SIZE`: 8192 字节
- `AGGREGATE_BUFFER_INITIAL_CAPACITY`: 8 KiB
- `MAX_EXEC_OUTPUT_DELTAS_PER_CALL`: 10,000

## 与其他模块的关系

- **unified_exec**：高级模块使用此模块的执行功能
- **spawn.rs**：使用 `spawn_child_async` 生成子进程
- **sandboxing**：使用 `SandboxManager`、`CommandSpec` 和 `ExecEnv`
- **bash.rs**、**shell.rs**：提供 shell 解析和配置
- **exec_env.rs**：使用环境变量创建功能
- **error.rs**：定义 `CodexErr` 和 `SandboxErr`
- **protocol.rs**：使用 `SandboxPolicy` 和事件类型
