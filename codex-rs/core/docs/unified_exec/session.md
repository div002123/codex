# unified_exec/session.rs

## 文件作用

管理 PTY 会话的生命周期和输出缓冲。提供会话的创建、输出收集、状态查询和沙箱拒绝检测功能。

## 主要结构体

### OutputBufferState
```rust
pub(crate) struct OutputBufferState {
    chunks: VecDeque<Vec<u8>>,
    pub(crate) total_bytes: usize,
}
```

输出缓冲区状态，使用双端队列存储输出块。

**主要方法**：
- `push_chunk(&mut self, chunk: Vec<u8>)`：添加输出块，自动清理超出限制的旧数据
- `drain(&mut self) -> Vec<Vec<u8>>`：提取并清空所有缓冲的数据
- `snapshot(&self) -> Vec<Vec<u8>>`：获取缓冲数据的快照（不清空）

**缓冲策略**：
- 当缓冲区超过 `UNIFIED_EXEC_OUTPUT_MAX_BYTES` (1 MiB) 时，自动删除最早的数据块
- 如果单个块的一部分超出限制，只删除超出的部分

### UnifiedExecSession
```rust
pub(crate) struct UnifiedExecSession {
    session: ExecCommandSession,
    output_buffer: OutputBuffer,
    output_notify: Arc<Notify>,
    output_task: JoinHandle<()>,
    sandbox_type: SandboxType,
}
```

统一执行会话，封装 PTY 会话和输出管理。

**主要方法**：
- `new(session, initial_output_rx, sandbox_type) -> Self`：创建新会话并启动输出收集任务
- `writer_sender() -> mpsc::Sender<Vec<u8>>`：获取用于写入标准输入的发送器
- `output_handles() -> OutputHandles`：获取输出缓冲区和通知句柄
- `has_exited() -> bool`：检查会话是否已退出
- `exit_code() -> Option<i32>`：获取退出码
- `snapshot_output() -> Vec<Vec<u8>>`：获取当前输出快照
- `sandbox_type() -> SandboxType`：获取沙箱类型
- `check_for_sandbox_denial() -> Result<(), UnifiedExecError>`：检查是否被沙箱拒绝
- `from_spawned(spawned, sandbox_type) -> Result<Self, UnifiedExecError>`：从生成的 PTY 创建会话

## 主要功能

### 输出收集
会话创建时启动后台任务持续收集输出：
```rust
tokio::spawn(async move {
    loop {
        match receiver.recv().await {
            Ok(chunk) => {
                let mut guard = buffer_clone.lock().await;
                guard.push_chunk(chunk);
                drop(guard);
                notify_clone.notify_waiters();
            }
            Err(tokio::sync::broadcast::error::RecvError::Lagged(_)) => continue,
            Err(tokio::sync::broadcast::error::RecvError::Closed) => break,
        }
    }
})
```

### 沙箱拒绝检测
使用启发式方法检测命令是否被沙箱拒绝：
1. 检查会话是否在沙箱模式下运行
2. 检查会话是否已退出
3. 等待短暂时间（20ms）以收集输出
4. 聚合所有输出块
5. 使用 `is_likely_sandbox_denied` 检查输出特征
6. 如果检测到拒绝，返回 `UnifiedExecError::SandboxDenied`

### 生命周期管理
实现 `Drop` trait 在会话被销毁时自动中止输出收集任务：
```rust
impl Drop for UnifiedExecSession {
    fn drop(&mut self) {
        self.output_task.abort();
    }
}
```

## 类型别名

```rust
pub(crate) type OutputBuffer = Arc<Mutex<OutputBufferState>>;
pub(crate) type OutputHandles = (OutputBuffer, Arc<Notify>);
```

- `OutputBuffer`：共享的线程安全输出缓冲区
- `OutputHandles`：输出缓冲区和通知句柄的元组

## 与其他模块的关系

- **mod.rs**：由主模块使用，提供会话管理能力
- **session_manager.rs**：被会话管理器调用以创建和管理会话
- **errors.rs**：使用 `UnifiedExecError` 报告错误
- **exec.rs**：使用 `is_likely_sandbox_denied` 和 `ExecToolCallOutput`
- **truncate.rs**：使用 `truncate_middle` 截断错误消息
- **codex_utils_pty**：依赖 PTY 工具库提供的 `ExecCommandSession` 和 `SpawnedPty`
