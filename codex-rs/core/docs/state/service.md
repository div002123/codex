# state/service.rs

## 文件作用

定义 `SessionServices` 结构体，集中管理会话级别的所有服务依赖和配置。

## 主要结构体

### SessionServices

会话服务容器，包含会话运行所需的所有外部服务和配置。

**字段：**

- `mcp_connection_manager: McpConnectionManager` - MCP 连接管理器
- `unified_exec_manager: UnifiedExecSessionManager` - 统一执行管理器
- `notifier: UserNotifier` - 用户通知服务
- `rollout: Mutex<Option<RolloutRecorder>>` - 滚动发布记录器（可选）
- `user_shell: crate::shell::Shell` - 用户 shell 配置
- `show_raw_agent_reasoning: bool` - 是否显示原始 agent 推理过程
- `auth_manager: Arc<AuthManager>` - 认证管理器
- `otel_event_manager: OtelEventManager` - OpenTelemetry 事件管理器
- `tool_approvals: Mutex<ApprovalStore>` - 工具审批存储

## 与其他模块的关系

- 被 `Session` 结构体持有，提供会话所需的所有服务
- 依赖多个外部模块：
  - `AuthManager` - 认证管理
  - `McpConnectionManager` - MCP 连接管理
  - `UnifiedExecSessionManager` - 执行管理
  - `UserNotifier` - 用户通知
  - `RolloutRecorder` - 功能发布追踪
  - `OtelEventManager` - 遥测事件管理
  - `ApprovalStore` - 工具审批管理
