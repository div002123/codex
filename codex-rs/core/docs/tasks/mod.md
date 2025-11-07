# tasks/mod.rs

## 文件作用

`tasks/mod.rs` 是任务系统的核心模块，定义了任务执行框架、任务生命周期管理和 `SessionTask` trait。

## 主要组件

### SessionTask Trait

定义异步任务的统一接口。

**方法：**

- `kind(&self) -> TaskKind` - 返回任务类型
- `async fn run(...)` - 执行任务直到完成或取消
- `async fn abort(...)` - 任务中止后的清理（默认为空操作）

### SessionTaskContext

会话任务上下文的轻量级包装。

**字段：**

- `session: Arc<Session>` - 会话引用

**方法：**

- `new(session: Arc<Session>) -> Self` - 创建上下文
- `clone_session(&self) -> Arc<Session>` - 克隆会话引用
- `auth_manager(&self) -> Arc<AuthManager>` - 获取认证管理器

### Session 扩展方法

为 `Session` 添加的任务管理方法：

- `async fn spawn_task<T: SessionTask>(...)` - 启动新任务
- `async fn abort_all_tasks(reason: TurnAbortReason)` - 中止所有任务
- `async fn on_task_finished(...)` - 任务完成处理
- `async fn register_new_active_task(...)` - 注册活跃任务
- `async fn take_all_running_tasks()` - 取出所有运行中的任务
- `async fn handle_task_abort(...)` - 处理任务中止

## 导出的任务类型

- `CompactTask` - 压缩任务
- `GhostSnapshotTask` - Git 快照任务
- `RegularTask` - 常规对话任务
- `ReviewTask` - 代码审查任务
- `UndoTask` - 撤销操作任务
- `UserShellCommandTask` - 用户 shell 命令任务

## 常量

- `GRACEFULL_INTERRUPTION_TIMEOUT_MS: u64 = 100` - 优雅中断超时时间（毫秒）

## 与其他模块的关系

- 定义 `SessionTask` trait，被所有具体任务实现
- 依赖 `Session` 和 `TurnContext` 提供执行环境
- 使用 `state` 模块的 `ActiveTurn`, `RunningTask`, `TaskKind`
- 与 `protocol` 模块交互，发送事件和处理用户输入
- 各子模块实现不同类型的具体任务
