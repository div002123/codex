# tasks/undo.rs

## 文件作用

实现 `UndoTask`，处理撤销操作，恢复之前的 Git 快照状态。

## 主要结构体

### UndoTask

撤销任务，恢复历史中最近的 Git ghost commit。

**构造：**

- `new() -> Self` - 创建新的撤销任务

## SessionTask 实现

### kind() -> TaskKind

返回 `TaskKind::Regular`。

### async fn run(...)

执行撤销流程：

**流程：**

1. 发送 `UndoStarted` 事件
2. 检查取消令牌
3. 从历史中查找最近的 `GhostSnapshot`
4. 在阻塞线程池中恢复 Git 快照
5. 成功时从历史中移除该快照
6. 发送 `UndoCompleted` 事件（包含成功/失败信息）

**错误处理：**

- 无快照可用：发送失败消息
- 恢复失败：记录警告/错误，发送失败消息
- 恢复成功：移除快照，更新历史

**返回：**

- `Option<String>` - 始终返回 None

## 与其他模块的关系

- 实现 `SessionTask` trait
- 使用 `codex_git::restore_ghost_commit` 恢复 Git 状态
- 与 `ContextManager` 交互：
  - 查找历史中的 `GhostSnapshot` 条目
  - 成功后移除快照条目
  - 更新会话历史
- 发送 `UndoStarted` 和 `UndoCompleted` 事件
- 在 `spawn_blocking` 中执行 Git 操作（避免阻塞异步运行时）
- 与 `GhostSnapshotTask` 配合，实现快照和恢复功能
