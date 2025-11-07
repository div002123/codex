# event_mapping.rs

## 文件作用

`event_mapping.rs` 实现 `ResponseItem` 到 `TurnItem` 的转换，用于解析和过滤会话历史中的消息。

## 主要函数

### parse_turn_item()
将 `ResponseItem` 解析为 `TurnItem`。

**参数:**
- `item: &ResponseItem` - 响应项

**返回:** `Option<TurnItem>` - 轮次项

**功能:**
- 解析用户消息（过滤会话前缀和用户指令）
- 解析助手消息
- 解析推理内容
- 解析工具调用和输出
- 解析 Web 搜索
- 过滤系统消息和其他无关项

### parse_user_message() (私有)
解析用户消息，过滤会话前缀。

### parse_agent_message() (私有)
解析助手消息。

### is_session_prefix() (私有)
检查是否为会话前缀消息（如 `<environment_context>`）。

## 与其他模块的关系

**依赖模块:**
- `codex_protocol::items::{TurnItem, UserMessageItem, AgentMessageItem, ...}` - 轮次项类型
- `codex_protocol::models::{ResponseItem, ContentItem, ...}` - 响应项类型
- `user_instructions::UserInstructions` - 用户指令检测

**被依赖模块:**
- `compact` - 收集用户消息时使用
- `conversation_manager` - 分叉会话时解析消息
- UI/CLI - 显示历史记录
