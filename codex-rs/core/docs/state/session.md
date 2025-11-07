# state/session.rs

## 文件作用

定义 `SessionState` 结构体，管理会话级别的可变状态，包括会话配置、对话历史和速率限制信息。

## 主要结构体

### SessionState

会话级别的持久化可变状态。

**字段：**

- `session_configuration: SessionConfiguration` - 会话配置
- `history: ContextManager` - 对话历史管理器
- `latest_rate_limits: Option<RateLimitSnapshot>` - 最新的速率限制快照

## 主要方法

### 构造方法

- `new(session_configuration: SessionConfiguration) -> Self` - 创建新的会话状态

### 历史管理

- `record_items<I>(&mut self, items: I)` - 记录对话条目
- `clone_history(&self) -> ContextManager` - 克隆历史记录
- `replace_history(&mut self, items: Vec<ResponseItem>)` - 替换历史记录

### Token 和速率限制管理

- `update_token_info_from_usage(&mut self, usage: &TokenUsage, model_context_window: Option<i64>)` - 更新 token 使用信息
- `token_info(&self) -> Option<TokenUsageInfo>` - 获取 token 使用信息
- `set_rate_limits(&mut self, snapshot: RateLimitSnapshot)` - 设置速率限制
- `token_info_and_rate_limits(&self) -> (Option<TokenUsageInfo>, Option<RateLimitSnapshot>)` - 获取 token 信息和速率限制
- `set_token_usage_full(&mut self, context_window: i64)` - 标记 token 使用已满

## 与其他模块的关系

- 依赖 `ContextManager` 管理对话历史
- 依赖 `SessionConfiguration` 存储会话配置
- 使用 `codex_protocol` 中的协议类型（ResponseItem, TokenUsage 等）
- 被 `Session` 结构体持有，提供会话状态管理
