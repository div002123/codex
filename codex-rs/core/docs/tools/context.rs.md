# context.rs

## 文件作用

本文件定义了工具调用的核心上下文类型和数据结构，为工具执行提供必要的环境信息、输入负载和输出格式化功能。它是工具系统的基础数据模型层。

## 主要类型别名

- **SharedTurnDiffTracker**: 共享的回合差异跟踪器（Arc<Mutex<TurnDiffTracker>>）

## 主要结构体

### 工具调用相关
- **ToolInvocation**: 工具调用上下文，包含：
  - session: 会话实例
  - turn: 回合上下文
  - tracker: 差异跟踪器
  - call_id: 调用 ID
  - tool_name: 工具名称
  - payload: 工具负载

- **ToolPayload**: 工具负载枚举，支持多种类型：
  - Function: 函数调用（带参数字符串）
  - Custom: 自定义工具调用（带输入字符串）
  - LocalShell: 本地 Shell 调用（带参数）
  - UnifiedExec: 统一执行（带参数字符串）
  - Mcp: MCP 工具调用（服务器、工具名、原始参数）

- **ToolOutput**: 工具输出枚举：
  - Function: 函数输出（内容、内容项、成功标志）
  - Mcp: MCP 输出（调用结果）

### 执行上下文相关
- **ExecCommandContext**: 执行命令上下文，包含：
  - turn: 回合上下文
  - call_id: 调用 ID
  - command_for_display: 显示用命令
  - cwd: 工作目录
  - apply_patch: 补丁应用上下文
  - tool_name: 工具名称
  - otel_event_manager: 遥测事件管理器
  - is_user_shell_command: 是否为用户 Shell 命令

- **ApplyPatchCommandContext**: 补丁应用上下文
  - user_explicitly_approved_this_action: 用户是否明确批准
  - changes: 文件变更映射

## 主要方法

### ToolPayload
- **log_payload()**: 返回用于日志记录的负载字符串

### ToolOutput
- **log_preview()**: 生成用于遥测的预览字符串
- **success_for_logging()**: 返回是否成功的布尔值
- **into_response()**: 将输出转换为响应项

## 辅助函数

- **telemetry_preview()**: 生成遥测预览，支持字节和行数限制，包含截断提示

## 与其他模块的关系

- 是工具系统的核心数据模型
- 被 `registry`、`router`、`orchestrator` 等模块使用
- 依赖 `Session` 和 `TurnContext` 提供执行环境
- 使用 `turn_diff_tracker` 模块跟踪文件变更
- 与 `codex_protocol` 定义的协议类型集成
- 支持 OTEL 遥测事件管理
