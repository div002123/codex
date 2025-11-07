# codex_delegate.rs

## 文件作用

`codex_delegate.rs` 实现了子代理（sub-agent）功能，允许在主会话中启动独立的 Codex 实例。子代理可以执行特定任务（如代码审查），并自动处理审批请求，同时将非审批事件转发给调用者。

## 核心功能

### 子代理模式
子代理是一个独立的 Codex 实例，运行在父会话的上下文中：
- 拥有独立的会话历史
- 继承父会话的配置
- 审批请求自动路由到父会话
- 其他事件转发给调用者

## 主要函数

### run_codex_conversation_interactive()
启动交互式子代理会话。

**参数:**
- `config: Config` - 配置
- `auth_manager: Arc<AuthManager>` - 认证管理器
- `parent_session: Arc<Session>` - 父会话
- `parent_ctx: Arc<TurnContext>` - 父轮次上下文
- `cancel_token: CancellationToken` - 取消令牌
- `initial_history: Option<InitialHistory>` - 初始历史（可选）

**返回:** `Result<Codex, CodexErr>` - 子代理 Codex 实例

**功能:**
1. 创建双向通道（操作和事件）
2. 使用 `SubAgentSource::Review` 启动新的 Codex 实例
3. 启动两个后台任务：
   - **事件转发任务:** 从子代理接收事件，过滤并处理审批请求，转发其他事件
   - **操作转发任务:** 从调用者接收操作，转发给子代理
4. 返回包装的 Codex 接口

**事件处理:**
- `AgentMessageDelta` / `AgentReasoningDelta` - 忽略（legacy事件）
- `SessionConfigured` - 忽略
- `ExecApprovalRequest` - 路由到 `handle_exec_approval()`
- `ApplyPatchApprovalRequest` - 路由到 `handle_patch_approval()`
- 其他事件 - 转发给调用者

### run_codex_conversation_one_shot()
启动一次性子代理会话。

**参数:**
- `config: Config` - 配置
- `auth_manager: Arc<AuthManager>` - 认证管理器
- `input: Vec<UserInput>` - 初始输入
- `parent_session: Arc<Session>` - 父会话
- `parent_ctx: Arc<TurnContext>` - 父轮次上下文
- `cancel_token: CancellationToken` - 取消令牌
- `initial_history: Option<InitialHistory>` - 初始历史

**返回:** `Result<Codex, CodexErr>` - 子代理 Codex 实例

**功能:**
1. 调用 `run_codex_conversation_interactive()` 创建子代理
2. 立即提交初始输入
3. 启动事件桥接任务：
   - 监听 `TaskComplete` 或 `TurnAborted` 事件
   - 自动发送 `Op::Shutdown` 并取消子代理
   - 转发所有事件给调用者
4. 返回只读的 Codex 接口（`tx_sub` 已关闭）

**适用场景:**
- 执行单次任务（如代码审查）
- 任务完成后自动清理
- 调用者不需要发送额外操作

### forward_events()
事件转发循环（内部函数）。

**参数:**
- `codex: Arc<Codex>` - 子代理实例
- `tx_sub: Sender<Event>` - 事件发送通道
- `parent_session: Arc<Session>` - 父会话
- `parent_ctx: Arc<TurnContext>` - 父上下文
- `cancel_token: CancellationToken` - 取消令牌

**功能:**
循环接收子代理事件，过滤并处理审批请求，转发其他事件。

### forward_ops()
操作转发循环（内部函数）。

**参数:**
- `codex: Arc<Codex>` - 子代理实例
- `rx_ops: Receiver<Submission>` - 操作接收通道
- `cancel_token_ops: CancellationToken` - 取消令牌

**功能:**
循环接收调用者操作，转发给子代理，直到取消或通道关闭。

### handle_exec_approval()
处理执行审批请求。

**参数:**
- `codex: &Codex` - 子代理实例
- `id: String` - 请求ID
- `parent_session: &Session` - 父会话
- `parent_ctx: &TurnContext` - 父上下文
- `event: ExecApprovalRequestEvent` - 审批请求事件
- `cancel_token: &CancellationToken` - 取消令牌

**功能:**
1. 调用父会话的 `request_command_approval()`
2. 使用 `await_approval_with_cancel()` 等待决定
3. 提交审批响应 `Op::ExecApproval`

### handle_patch_approval()
处理补丁审批请求。

**参数:**
- `codex: &Codex` - 子代理实例
- `id: String` - 请求ID
- `parent_session: &Session` - 父会话
- `parent_ctx: &TurnContext` - 父上下文
- `event: ApplyPatchApprovalRequestEvent` - 审批请求事件
- `cancel_token: &CancellationToken` - 取消令牌

**功能:**
1. 调用父会话的 `request_patch_approval()`
2. 使用 `await_approval_with_cancel()` 等待决定
3. 提交审批响应 `Op::PatchApproval`

### await_approval_with_cancel()
等待审批决定，支持取消。

**参数:**
- `fut: F` - 审批Future
- `parent_session: &Session` - 父会话
- `sub_id: &str` - 子代理ID
- `cancel_token: &CancellationToken` - 取消令牌

**返回:** `ReviewDecision` - 审批决定

**功能:**
使用 `tokio::select!` 竞争等待：
- 取消令牌触发 → 通知父会话并返回 `ReviewDecision::Abort`
- 审批完成 → 返回决定

## 与其他模块的关系

**依赖模块:**
- `codex::Codex` - 核心 Codex 实例
- `codex::Session` - 父会话管理
- `codex::TurnContext` - 轮次上下文
- `config::Config` - 配置管理
- `AuthManager` - 认证管理
- `protocol::{Event, Op, Submission, ReviewDecision}` - 协议类型

**被依赖模块:**
- `tasks::ReviewTask` - 代码审查任务使用子代理执行审查

**核心流程:**
1. 任务触发（如 /review 命令）
2. 创建子代理配置
3. 调用 `run_codex_conversation_one_shot()` 或 `run_codex_conversation_interactive()`
4. 子代理执行任务
5. 审批请求自动路由到父会话
6. 其他事件转发给任务处理器
7. 任务完成后自动清理

## 设计模式

### 审批路由
子代理不直接与用户交互，所有审批请求都路由到父会话：
- 保持单一审批流程
- 用户体验一致
- 父会话完全控制权限

### 自动清理
一次性子代理在任务完成后自动关闭：
- 监听 `TaskComplete` / `TurnAborted` 事件
- 发送 `Op::Shutdown`
- 取消子代理令牌
- 释放资源

### 取消传播
- 父取消令牌 → 子令牌自动取消
- 子令牌取消不影响父令牌
- 每个任务有独立的取消作用域

## 常量

- `SUBMISSION_CHANNEL_CAPACITY` - 提交通道容量（继承自 `codex::SUBMISSION_CHANNEL_CAPACITY`）
