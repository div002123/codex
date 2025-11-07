# state/turn.rs

## 文件作用

定义轮次（Turn）级别的状态和元数据结构，管理当前活跃的任务和轮次内的可变状态。

## 主要结构体和枚举

### ActiveTurn

当前运行的轮次的元数据容器。

**字段：**

- `tasks: IndexMap<String, RunningTask>` - 正在运行的任务映射（sub_id -> task）
- `turn_state: Arc<Mutex<TurnState>>` - 轮次可变状态

**主要方法：**

- `add_task(&mut self, task: RunningTask)` - 添加任务
- `remove_task(&mut self, sub_id: &str) -> bool` - 移除任务，返回是否所有任务都已完成
- `drain_tasks(&mut self) -> Vec<RunningTask>` - 取出所有任务
- `clear_pending(&self)` - 清除待处理的审批和输入

### TaskKind

任务类型枚举。

**变体：**

- `Regular` - 常规任务
- `Review` - 审查任务
- `Compact` - 压缩任务

### RunningTask

正在运行的任务信息。

**字段：**

- `done: Arc<Notify>` - 完成通知
- `kind: TaskKind` - 任务类型
- `task: Arc<dyn SessionTask>` - 任务实现
- `cancellation_token: CancellationToken` - 取消令牌
- `handle: Arc<AbortOnDropHandle<()>>` - 任务句柄
- `turn_context: Arc<TurnContext>` - 轮次上下文

### TurnState

单个轮次的可变状态。

**字段：**

- `pending_approvals: HashMap<String, oneshot::Sender<ReviewDecision>>` - 待审批映射
- `pending_input: Vec<ResponseInputItem>` - 待处理输入

**主要方法：**

- `insert_pending_approval(&mut self, key: String, tx: oneshot::Sender<ReviewDecision>)` - 插入待审批
- `remove_pending_approval(&mut self, key: &str)` - 移除待审批
- `clear_pending(&mut self)` - 清除所有待处理项
- `push_pending_input(&mut self, input: ResponseInputItem)` - 添加待处理输入
- `take_pending_input(&mut self) -> Vec<ResponseInputItem>` - 取出所有待处理输入

## 与其他模块的关系

- 被 `Session` 使用，管理当前活跃的轮次和任务
- 依赖 `tasks::SessionTask` trait 定义任务接口
- 依赖 `TurnContext` 提供轮次上下文信息
- 使用 `tokio` 的异步原语（Notify, CancellationToken, Mutex 等）
- 与 `protocol` 模块交互，处理审查决策和输入
