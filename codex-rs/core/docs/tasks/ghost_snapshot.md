# tasks/ghost_snapshot.rs

## 文件作用

实现 `GhostSnapshotTask`，在后台创建 Git ghost commit 快照，用于撤销操作。

## 主要结构体

### GhostSnapshotTask

后台 Git 快照任务，记录工作区状态以支持撤销。

**字段：**

- `token: Token` - 就绪令牌（用于同步工具调用门控）

**构造：**

- `new(token: Token) -> Self` - 创建快照任务

## SessionTask 实现

### kind() -> TaskKind

返回 `TaskKind::Regular`。

### async fn run(...)

执行快照创建流程：

**流程：**

1. 在新的 tokio 任务中运行（非阻塞）
2. 使用 `spawn_blocking` 在阻塞线程池中创建 ghost commit
3. 成功时：
   - 记录日志
   - 将 `GhostSnapshot` 条目添加到对话历史
4. 失败时：
   - 记录警告
   - 根据错误类型发送用户通知：
     - `NotAGitRepository`: 提示当前目录不是 Git 仓库
     - 其他错误：禁用快照并报告错误
5. 完成后标记就绪门控（无论成功或失败）

**取消处理：**

- 使用 `tokio::select!` 监听取消令牌
- 取消时记录日志并标记门控就绪

**返回：**

- `Option<String>` - 始终返回 None

## 与其他模块的关系

- 实现 `SessionTask` trait
- 使用 `codex_git::create_ghost_commit` 创建快照
- 将 `ResponseItem::GhostSnapshot` 记录到会话历史
- 与 `UndoTask` 配合，提供撤销功能
- 使用 `codex_utils_readiness::Token` 控制工具调用时机
- 在独立的 tokio 任务中运行，不阻塞主任务流程
- 错误时通过 `Session::notify_background_event` 通知用户
- 使用 `CreateGhostCommitOptions` 配置快照选项
