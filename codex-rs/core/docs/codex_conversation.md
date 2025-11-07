# codex_conversation.rs

## 文件作用

`codex_conversation.rs` 提供了 `CodexConversation` 结构体，作为双向消息流的接口，封装了 Codex 会话的基本操作。它是客户端与 Codex 核心交互的主要入口点。

## 主要结构体

### CodexConversation
Codex 对话的接口封装。

**字段:**
- `codex: Codex` - 底层 Codex 实例
- `rollout_path: PathBuf` - rollout 文件路径（用于持久化会话历史）

**方法:**

#### new()
创建新的会话实例（内部使用）。

**参数:**
- `codex: Codex` - Codex 实例
- `rollout_path: PathBuf` - rollout 路径

**返回:** `Self`

#### submit()
提交操作到 Codex。

**参数:**
- `op: Op` - 操作（用户输入、审批响应等）

**返回:** `CodexResult<String>` - 提交ID

**功能:**
将操作提交到底层 Codex 实例，返回唯一提交ID用于追踪。

#### submit_with_id()
使用指定ID提交操作（谨慎使用，计划移除）。

**参数:**
- `sub: Submission` - 包含ID和操作的提交

**返回:** `CodexResult<()>`

**注意:** 这个方法主要用于内部或特殊场景，一般应使用 `submit()`。

#### next_event()
获取下一个事件。

**返回:** `CodexResult<Event>` - 下一个事件

**功能:**
从 Codex 接收下一个事件（如模型响应、工具调用、错误等）。这是一个异步阻塞调用，会等待直到有事件可用。

#### rollout_path()
获取 rollout 路径。

**返回:** `PathBuf` - rollout 文件路径

**功能:**
返回会话历史持久化文件的路径，用于恢复或导出会话。

## 与其他模块的关系

**依赖模块:**
- `codex::Codex` - 核心 Codex 实例
- `protocol::{Event, Op, Submission}` - 协议定义

**被依赖模块:**
- `conversation_manager::ConversationManager` - 多会话管理器，创建和管理 CodexConversation 实例

**核心流程:**
1. ConversationManager 创建 CodexConversation
2. 客户端通过 submit() 发送操作
3. Codex 处理操作
4. 客户端通过 next_event() 接收事件
5. 循环直到会话结束

## 使用示例

```rust
// 提交用户输入
conversation.submit(Op::UserInput { items: vec![...] }).await?;

// 接收事件
loop {
    let event = conversation.next_event().await?;
    match event.msg {
        EventMsg::AgentMessage(msg) => println!("{}", msg.message),
        EventMsg::TaskComplete(_) => break,
        // ... 处理其他事件
    }
}
```

## 设计说明

`CodexConversation` 是一个轻量级的封装层，主要目的是：
1. 提供清晰的会话级别接口
2. 封装 rollout 路径管理
3. 简化客户端代码
4. 为未来的会话级功能预留扩展空间
