# tasks/compact.rs

## 文件作用

实现 `CompactTask`，处理对话历史压缩任务，用于管理上下文窗口。

## 主要结构体

### CompactTask

对话历史压缩任务，在上下文窗口接近满载时压缩历史记录。

**特点：**

- 无状态结构体（可复制、可默认构造）
- 实现 `SessionTask` trait
- 任务类型为 `TaskKind::Compact`

## SessionTask 实现

### kind() -> TaskKind

返回 `TaskKind::Compact`。

### async fn run(...)

执行压缩任务流程：

**参数：**

- `session: Arc<SessionTaskContext>` - 会话上下文
- `ctx: Arc<TurnContext>` - 轮次上下文
- `input: Vec<UserInput>` - 用户输入
- `_cancellation_token: CancellationToken` - 取消令牌（未使用）

**返回：**

- `Option<String>` - 可选的最终消息

**实现：**

调用 `crate::codex::compact::run_compact_task()` 执行压缩逻辑。

## 与其他模块的关系

- 实现 `SessionTask` trait
- 委托给 `crate::codex::compact::run_compact_task` 处理压缩逻辑
- 与 `ContextManager` 协作，管理对话历史
- 当上下文窗口接近满载时自动触发
- 帮助保持模型输入在可接受的令牌限制内
