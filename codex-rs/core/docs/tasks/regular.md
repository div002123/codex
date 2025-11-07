# tasks/regular.rs

## 文件作用

实现 `RegularTask`，处理常规的对话任务，是最基础的任务类型。

## 主要结构体

### RegularTask

常规对话任务，执行标准的用户输入处理流程。

**特点：**

- 无状态结构体（可复制、可默认构造）
- 实现 `SessionTask` trait
- 任务类型为 `TaskKind::Regular`

## SessionTask 实现

### kind() -> TaskKind

返回 `TaskKind::Regular`。

### async fn run(...)

执行常规任务流程：

**参数：**

- `session: Arc<SessionTaskContext>` - 会话上下文
- `ctx: Arc<TurnContext>` - 轮次上下文
- `input: Vec<UserInput>` - 用户输入
- `cancellation_token: CancellationToken` - 取消令牌

**返回：**

- `Option<String>` - 可选的最终 agent 消息

**实现：**

调用 `crate::codex::run_task()` 执行标准对话流程。

## 与其他模块的关系

- 实现 `SessionTask` trait
- 委托给 `crate::codex::run_task` 处理实际逻辑
- 被 `Session::spawn_task` 使用，处理用户的常规对话请求
- 是最常用的任务类型，处理日常的对话交互
