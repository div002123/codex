# conversation_manager.rs

## 文件作用

`conversation_manager.rs` 实现了多会话管理功能，负责创建、存储、检索和管理多个 Codex 会话。它提供了会话的生命周期管理，包括创建新会话、恢复已有会话、分叉会话等功能。

## 主要结构体

### ConversationManager
会话管理器，维护内存中的会话集合。

**字段:**
- `conversations: Arc<RwLock<HashMap<ConversationId, Arc<CodexConversation>>>>` - 会话映射表
- `auth_manager: Arc<AuthManager>` - 认证管理器
- `session_source: SessionSource` - 会话来源标识

**方法:**

#### new()
创建新的会话管理器。

**参数:**
- `auth_manager: Arc<AuthManager>` - 认证管理器
- `session_source: SessionSource` - 会话来源（CLI、VSCode、Exec等）

**返回:** `Self`

#### with_auth()
使用指定认证信息创建管理器（仅用于测试）。

**参数:**
- `auth: CodexAuth` - 认证信息

**返回:** `Self`

**注意:** 仅用于集成测试，不应在业务逻辑中使用。

#### new_conversation()
创建新会话。

**参数:**
- `config: Config` - 会话配置

**返回:** `CodexResult<NewConversation>` - 新会话信息

**功能:**
1. 使用配置和认证管理器启动新的 Codex 实例
2. 生成唯一的 `ConversationId`
3. 接收并验证第一个事件（必须是 `SessionConfigured`）
4. 创建 `CodexConversation` 封装
5. 存储到内存映射表
6. 返回会话信息

#### get_conversation()
获取已有会话。

**参数:**
- `conversation_id: ConversationId` - 会话ID

**返回:** `CodexResult<Arc<CodexConversation>>` - 会话引用

**错误:**
- `CodexErr::ConversationNotFound` - 会话不存在

#### resume_conversation_from_rollout()
从 rollout 文件恢复会话。

**参数:**
- `config: Config` - 会话配置
- `rollout_path: PathBuf` - rollout 文件路径
- `auth_manager: Arc<AuthManager>` - 认证管理器

**返回:** `CodexResult<NewConversation>` - 恢复的会话

**功能:**
1. 从 rollout 文件读取历史记录
2. 使用历史记录创建 `InitialHistory::Resumed`
3. 调用 `resume_conversation_with_history()`

#### resume_conversation_with_history()
使用指定历史恢复会话。

**参数:**
- `config: Config` - 会话配置
- `initial_history: InitialHistory` - 初始历史
- `auth_manager: Arc<AuthManager>` - 认证管理器

**返回:** `CodexResult<NewConversation>` - 恢复的会话

**功能:**
1. 使用初始历史启动 Codex
2. 调用 `finalize_spawn()` 完成初始化

#### remove_conversation()
移除会话。

**参数:**
- `conversation_id: &ConversationId` - 会话ID

**返回:** `Option<Arc<CodexConversation>>` - 被移除的会话（如果存在）

**注意:**
- 会话使用 `Arc` 共享，可能在其他地方仍有引用
- 仅从管理器的映射表中移除

#### fork_conversation()
分叉已有会话。

**参数:**
- `nth_user_message: usize` - 分叉点（第n条用户消息之前，0-based）
- `config: Config` - 新会话配置
- `path: PathBuf` - 源会话的 rollout 路径

**返回:** `CodexResult<NewConversation>` - 新的分叉会话

**功能:**
1. 从 rollout 读取历史
2. 使用 `truncate_before_nth_user_message()` 截断到指定点
3. 使用截断后的历史创建新会话
4. 新会话拥有独立的 `ConversationId`

**使用场景:**
- 回退到之前的对话状态
- 探索不同的对话路径
- 从特定点开始新的对话分支

#### spawn_conversation() (私有)
启动新会话的内部方法。

#### finalize_spawn() (私有)
完成会话启动的内部方法，验证并存储会话。

### NewConversation
新创建会话的信息。

**字段:**
- `conversation_id: ConversationId` - 会话ID
- `conversation: Arc<CodexConversation>` - 会话对象
- `session_configured: SessionConfiguredEvent` - 会话配置事件

## 主要函数

### truncate_before_nth_user_message()
截断历史到第n条用户消息之前。

**参数:**
- `history: InitialHistory` - 原始历史
- `n: usize` - 用户消息索引（0-based）

**返回:** `InitialHistory` - 截断后的历史

**功能:**
1. 提取 rollout 项列表
2. 找到所有用户消息的位置
3. 截断到第n条用户消息之前（不包括第n条）
4. 排除会话前缀消息（AGENTS.md、ENVIRONMENT_CONTEXT等）
5. 如果索引越界或结果为空，返回 `InitialHistory::New`

**测试覆盖:**
- 正确截断到指定用户消息
- 忽略会话前缀消息
- 处理边界条件

## 与其他模块的关系

**依赖模块:**
- `codex::Codex` - 核心 Codex 实例
- `codex_conversation::CodexConversation` - 会话封装
- `config::Config` - 配置管理
- `rollout::RolloutRecorder` - 历史记录持久化
- `AuthManager` - 认证管理
- `protocol::{InitialHistory, SessionSource}` - 协议类型
- `event_mapping::parse_turn_item` - 事件解析

**被依赖模块:**
- `main` / CLI / API服务器 - 使用管理器创建和管理会话
- VSCode扩展 - 通过管理器管理多个会话

**核心流程:**

**创建新会话:**
```
客户端 → new_conversation() → spawn() → finalize_spawn() → 存储到映射表
```

**恢复会话:**
```
客户端 → resume_conversation_from_rollout() → 读取rollout → resume_conversation_with_history() → 存储到映射表
```

**分叉会话:**
```
客户端 → fork_conversation() → 读取rollout → 截断历史 → spawn() → 新会话
```

## 设计说明

### 会话隔离
每个会话拥有：
- 独立的 `ConversationId`
- 独立的历史记录
- 独立的 rollout 文件
- 共享的配置和认证（通过 `Arc`）

### 并发安全
- 使用 `RwLock` 保护会话映射表
- 支持多个读取器同时访问
- 写操作（添加/移除）互斥

### 内存管理
- 会话使用 `Arc<CodexConversation>` 共享所有权
- 客户端可以持有会话引用
- 从管理器移除不会立即销毁会话（如果有其他引用）

### 首事件验证
所有会话启动后，第一个事件必须是 `SessionConfigured`：
- 确保会话正确初始化
- 提供会话元数据（rollout路径、模型信息等）
- 客户端可以显示在历史记录中

## 测试

包含单元测试验证：
- 正确截断到指定用户消息之前
- 忽略会话前缀消息（AGENTS.md、ENVIRONMENT_CONTEXT）
- 处理多个用户消息和助手消息
- 处理边界条件（越界、空历史）
