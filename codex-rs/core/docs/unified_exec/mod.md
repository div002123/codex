# unified_exec/mod.rs

## 文件作用

Unified Exec 是一个交互式 PTY 执行编排模块，负责管理带有审批和沙箱功能的交互式 PTY 会话。

该模块是命令执行的统一入口，整合了审批流程、沙箱选择和重试语义，提供了一个描述性清晰的执行流程。

## 主要功能

1. **会话管理**：创建、复用和管理交互式 PTY 会话
2. **输出缓冲**：缓冲会话输出并设置容量限制
3. **审批编排**：使用 ToolOrchestrator 处理审批、沙箱选择和重试语义
4. **沙箱重试**：沙箱拒绝时自动使用无沙箱模式重试
5. **沙箱拒绝检测**：使用共享的 `is_likely_sandbox_denied` 启发式判断

## 执行流程

1. 构建请求 `{ command, cwd }`
2. 编排器处理：审批（跳过/缓存/提示）→ 选择沙箱 → 运行
3. 运行时：转换 `CommandSpec` → `ExecEnv` → 生成 PTY
4. 如果被沙箱拒绝，编排器使用 `SandboxType::None` 重试
5. 返回带有流式输出和元数据的会话

## 主要结构体

### UnifiedExecContext
```rust
pub(crate) struct UnifiedExecContext {
    pub session: Arc<Session>,
    pub turn: Arc<TurnContext>,
    pub call_id: String,
}
```
执行上下文，包含会话、轮次和调用 ID。

### ExecCommandRequest
```rust
pub(crate) struct ExecCommandRequest<'a> {
    pub command: &'a str,
    pub shell: &'a str,
    pub login: bool,
    pub yield_time_ms: Option<u64>,
    pub max_output_tokens: Option<usize>,
}
```
执行命令请求参数。

### WriteStdinRequest
```rust
pub(crate) struct WriteStdinRequest<'a> {
    pub session_id: i32,
    pub input: &'a str,
    pub yield_time_ms: Option<u64>,
    pub max_output_tokens: Option<usize>,
}
```
向标准输入写入数据的请求参数。

### UnifiedExecResponse
```rust
pub(crate) struct UnifiedExecResponse {
    pub event_call_id: String,
    pub chunk_id: String,
    pub wall_time: Duration,
    pub output: String,
    pub session_id: Option<i32>,
    pub exit_code: Option<i32>,
    pub original_token_count: Option<usize>,
}
```
执行响应，包含输出、会话 ID、退出码等信息。

### UnifiedExecSessionManager
```rust
pub(crate) struct UnifiedExecSessionManager {
    next_session_id: AtomicI32,
    sessions: Mutex<HashMap<i32, SessionEntry>>,
}
```
会话管理器，负责创建和管理所有活动的 PTY 会话。

## 主要函数

### clamp_yield_time
```rust
pub(crate) fn clamp_yield_time(yield_time_ms: Option<u64>) -> u64
```
将等待时间限制在 `MIN_YIELD_TIME_MS` 到 `MAX_YIELD_TIME_MS` 之间。

### resolve_max_tokens
```rust
pub(crate) fn resolve_max_tokens(max_tokens: Option<usize>) -> usize
```
解析最大 token 数量，默认为 `DEFAULT_MAX_OUTPUT_TOKENS`。

### generate_chunk_id
```rust
pub(crate) fn generate_chunk_id() -> String
```
生成随机的 6 字符十六进制块 ID。

### truncate_output_to_tokens
```rust
pub(crate) fn truncate_output_to_tokens(
    output: &str,
    max_tokens: usize,
) -> (String, Option<usize>)
```
将输出截断到指定的 token 数量，保留前半部分和后半部分，中间显示截断信息。

## 常量配置

- `DEFAULT_YIELD_TIME_MS`: 10,000 ms（默认等待时间）
- `MIN_YIELD_TIME_MS`: 250 ms（最小等待时间）
- `MAX_YIELD_TIME_MS`: 30,000 ms（最大等待时间）
- `DEFAULT_MAX_OUTPUT_TOKENS`: 10,000（默认最大输出 token）
- `UNIFIED_EXEC_OUTPUT_MAX_BYTES`: 1 MiB（输出最大字节数）

## 与其他模块的关系

- **session.rs**：提供 PTY 会话生命周期和输出缓冲
- **session_manager.rs**：提供编排逻辑（审批、沙箱、复用）和请求处理
- **errors.rs**：定义统一执行的错误类型
- **exec.rs**：使用基础执行功能和沙箱检测
- **sandboxing**：提供沙箱转换和执行环境
- **tools/orchestrator**：使用工具编排器处理审批和沙箱
- **codex_utils_pty**：使用 PTY 进程生成功能
