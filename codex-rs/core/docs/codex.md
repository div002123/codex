# codex.rs

## 文件作用

`codex.rs` 是 Codex 系统的核心模块，实现了整个 AI 代理的主要业务逻辑。它负责管理会话（Session）、处理用户输入、协调模型交互、工具调用、审批流程等核心功能。

## 主要结构体

### Codex
高层接口，作为队列对，发送提交（Submission）并接收事件（Event）。

**字段:**
- `next_id: AtomicU64` - 生成唯一提交ID的计数器
- `tx_sub: Sender<Submission>` - 提交通道发送端
- `rx_event: Receiver<Event>` - 事件通道接收端

**方法:**
- `spawn()` - 创建新的 Codex 实例并初始化会话
- `submit()` - 提交操作并生成唯一ID
- `submit_with_id()` - 使用指定ID提交操作
- `next_event()` - 获取下一个事件

### Session
会话上下文，管理会话状态、历史记录、工具路由等。

**字段:**
- `conversation_id: ConversationId` - 唯一会话标识符
- `tx_event: Sender<Event>` - 事件发送通道
- `state: Mutex<SessionState>` - 会话状态
- `active_turn: Mutex<Option<ActiveTurn>>` - 当前活动轮次
- `services: SessionServices` - 会话服务（MCP连接、工具路由等）
- `next_internal_sub_id: AtomicU64` - 内部提交ID计数器

**核心方法:**
- `new()` - 创建新会话
- `make_turn_context()` - 为新轮次创建上下文
- `record_into_history()` - 记录响应项到历史
- `clone_history()` - 克隆会话历史
- `replace_history()` - 替换会话历史
- `build_initial_context()` - 构建初始上下文（系统提示）
- `request_command_approval()` - 请求命令执行审批
- `request_patch_approval()` - 请求补丁应用审批
- `send_event()` - 发送事件到客户端
- `persist_rollout_items()` - 持久化rollout项

### TurnContext
单轮对话的上下文信息。

**字段:**
- `sub_id: String` - 提交ID
- `client: ModelClient` - 模型客户端
- `cwd: PathBuf` - 当前工作目录
- `developer_instructions: Option<String>` - 开发者指令
- `base_instructions: Option<String>` - 基础指令
- `compact_prompt: Option<String>` - 压缩提示
- `user_instructions: Option<String>` - 用户指令
- `approval_policy: AskForApproval` - 审批策略
- `sandbox_policy: SandboxPolicy` - 沙箱策略
- `shell_environment_policy: ShellEnvironmentPolicy` - Shell环境策略
- `tools_config: ToolsConfig` - 工具配置
- `final_output_json_schema: Option<Value>` - 最终输出的JSON schema
- `codex_linux_sandbox_exe: Option<PathBuf>` - Linux沙箱可执行文件路径
- `tool_call_gate: Arc<ReadinessFlag>` - 工具调用就绪标志

**方法:**
- `resolve_path()` - 解析相对于 cwd 的路径
- `compact_prompt()` - 获取压缩提示

### SessionConfiguration
会话配置信息。

**字段:**
- `provider: ModelProviderInfo` - 模型提供商
- `model: String` - 模型名称
- `model_reasoning_effort: Option<ReasoningEffortConfig>` - 推理强度配置
- `model_reasoning_summary: ReasoningSummaryConfig` - 推理摘要配置
- `developer_instructions: Option<String>` - 开发者指令
- `user_instructions: Option<String>` - 用户指令
- `base_instructions: Option<String>` - 基础指令覆盖
- `compact_prompt: Option<String>` - 压缩提示覆盖
- `approval_policy: AskForApproval` - 审批策略
- `sandbox_policy: SandboxPolicy` - 沙箱策略
- `cwd: PathBuf` - 工作目录
- `features: Features` - 功能标志集
- `original_config_do_not_use: Arc<Config>` - 原始配置（内部使用）
- `session_source: SessionSource` - 会话来源

**方法:**
- `apply()` - 应用会话设置更新

### CodexSpawnOk
Codex spawn 操作的返回结果。

**字段:**
- `codex: Codex` - 创建的 Codex 实例
- `conversation_id: ConversationId` - 唯一会话ID

## 主要函数

### submission_loop()
提交处理循环，处理用户输入和操作。主要逻辑包括：
- 接收并路由各种操作（用户输入、审批响应、配置更新等）
- 管理会话任务的启动和中断
- 处理会话关闭和清理

## 与其他模块的关系

**依赖模块:**
- `client::ModelClient` - 与AI模型通信
- `tools::ToolRouter` - 工具调用路由
- `mcp_connection_manager::McpConnectionManager` - MCP服务器连接管理
- `rollout::RolloutRecorder` - 会话记录持久化
- `context_manager::ContextManager` - 上下文窗口管理
- `tasks::SessionTask` - 会话任务（Review、GhostSnapshot等）
- `unified_exec::UnifiedExecSessionManager` - 统一执行管理器

**被依赖模块:**
- `codex_conversation::CodexConversation` - 会话接口封装
- `conversation_manager::ConversationManager` - 多会话管理
- `codex_delegate` - 子代理模块

**核心流程:**
1. 客户端创建 Codex 实例并初始化会话
2. 提交用户输入或操作
3. Session 处理提交，创建 TurnContext
4. 调用模型生成响应
5. 处理工具调用、审批请求等
6. 记录历史并持久化
7. 发送事件到客户端

## 常量

- `INITIAL_SUBMIT_ID` - 初始提交ID（空字符串）
- `SUBMISSION_CHANNEL_CAPACITY` - 提交通道容量（64）
