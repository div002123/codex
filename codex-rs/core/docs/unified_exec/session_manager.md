# unified_exec/session_manager.rs

## 文件作用

实现会话管理器的核心编排逻辑，负责处理命令执行请求、标准输入写入、会话生命周期管理、输出收集和事件发送。

## 主要功能

### 命令执行（exec_command）
```rust
pub(crate) async fn exec_command(
    &self,
    request: ExecCommandRequest<'_>,
    context: &UnifiedExecContext,
) -> Result<UnifiedExecResponse, UnifiedExecError>
```

执行流程：
1. 构建 shell 命令（如 `bash -lc "command"`）
2. 通过 `open_session_with_sandbox` 打开带沙箱的会话
3. 等待指定时间（yield_time_ms）收集输出
4. 截断输出到指定 token 数量
5. 如果命令已完成，清理会话并发送完成事件
6. 如果命令仍在运行，存储会话并返回 session_id

### 标准输入写入（write_stdin）
```rust
pub(crate) async fn write_stdin(
    &self,
    request: WriteStdinRequest<'_>,
) -> Result<UnifiedExecResponse, UnifiedExecError>
```

写入流程：
1. 准备会话句柄（writer_tx, output_buffer, output_notify）
2. 向会话写入输入数据（如果非空）
3. 等待 100ms 让命令处理输入
4. 收集指定时间内的输出
5. 刷新会话状态，检查是否已完成
6. 如果完成，清理会话并发送完成事件

### 会话打开与沙箱编排
```rust
async fn open_session_with_sandbox(
    &self,
    command: Vec<String>,
    context: &UnifiedExecContext,
) -> Result<UnifiedExecSession, UnifiedExecError>
```

使用 `ToolOrchestrator` 编排：
1. 创建 `UnifiedExecRequest` 和 `ToolCtx`
2. 调用编排器的 `run` 方法
3. 编排器处理审批、沙箱选择和重试
4. 最终调用 `open_session_with_exec_env` 生成 PTY

### 直接会话打开
```rust
pub(crate) async fn open_session_with_exec_env(
    &self,
    env: &ExecEnv,
) -> Result<UnifiedExecSession, UnifiedExecError>
```

从 `ExecEnv` 生成 PTY 会话：
1. 解析命令和参数
2. 使用 `codex_utils_pty::spawn_pty_process` 生成 PTY
3. 通过 `UnifiedExecSession::from_spawned` 封装会话

## 辅助方法

### refresh_session_state
```rust
async fn refresh_session_state(&self, session_id: i32) -> SessionStatus
```
刷新会话状态并返回：
- `SessionStatus::Alive`：会话仍在运行
- `SessionStatus::Exited`：会话已退出（并从管理器中移除）
- `SessionStatus::Unknown`：未知会话 ID

### prepare_session_handles
```rust
async fn prepare_session_handles(
    &self,
    session_id: i32,
) -> Result<(mpsc::Sender<Vec<u8>>, OutputBuffer, Arc<Notify>), UnifiedExecError>
```
获取会话的输入写入器、输出缓冲区和通知句柄。

### send_input
```rust
async fn send_input(
    writer_tx: &mpsc::Sender<Vec<u8>>,
    data: &[u8],
) -> Result<(), UnifiedExecError>
```
向会话的标准输入发送数据。

### store_session
```rust
async fn store_session(
    &self,
    session: UnifiedExecSession,
    context: &UnifiedExecContext,
    command: &str,
    started_at: Instant,
) -> i32
```
存储活动会话并分配唯一 ID。

### collect_output_until_deadline
```rust
pub(super) async fn collect_output_until_deadline(
    output_buffer: &OutputBuffer,
    output_notify: &Arc<Notify>,
    deadline: Instant,
) -> Vec<u8>
```
循环收集输出直到截止时间：
1. 提取缓冲区中的所有数据块
2. 如果缓冲区为空，等待通知或超时
3. 将收集的数据块合并为单个字节向量
4. 在截止时间到达时返回

## 事件发送

### emit_exec_end_from_entry
```rust
async fn emit_exec_end_from_entry(
    entry: SessionEntry,
    aggregated_output: String,
    exit_code: i32,
    duration: Duration,
)
```
从会话条目发送执行完成事件。

### emit_exec_end_from_context
```rust
async fn emit_exec_end_from_context(
    context: &UnifiedExecContext,
    command: String,
    aggregated_output: String,
    exit_code: i32,
    duration: Duration,
)
```
从上下文发送执行完成事件。

## 内部枚举

### SessionStatus
```rust
enum SessionStatus {
    Alive { exit_code: Option<i32>, call_id: String },
    Exited { exit_code: Option<i32>, entry: Box<SessionEntry> },
    Unknown,
}
```
表示会话状态的内部枚举。

## 与其他模块的关系

- **mod.rs**：实现 `UnifiedExecSessionManager` 的方法
- **session.rs**：使用 `UnifiedExecSession` 管理 PTY 会话
- **errors.rs**：返回 `UnifiedExecError`
- **exec.rs**：使用 `ExecToolCallOutput` 和 `StreamOutput`
- **exec_env.rs**：使用 `create_env` 创建环境变量
- **sandboxing**：使用 `ExecEnv` 执行环境
- **tools/orchestrator**：使用 `ToolOrchestrator` 编排审批和沙箱
- **tools/runtimes/unified_exec**：使用 `UnifiedExecRequest` 和 `UnifiedExecRuntime`
- **tools/events**：使用 `ToolEmitter` 和 `ToolEventCtx` 发送事件
- **codex_utils_pty**：使用 `spawn_pty_process` 生成 PTY 进程
