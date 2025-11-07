# rollout/recorder.rs

## 文件作用

`rollout/recorder.rs` 实现 rollout 记录器，负责将会话历史持久化到文件系统。

## 主要结构体

### RolloutRecorder
Rollout 记录器。

**方法:**
- `new()` - 创建记录器
- `record()` - 记录 rollout 项
- `get_rollout_history()` - 读取 rollout 历史

### RolloutRecorderParams
记录器参数。

**字段:**
- `conversation_id: ConversationId` - 会话 ID
- `user_instructions: Option<String>` - 用户指令
- `session_source: SessionSource` - 会话来源

**方法:**
- `new()` - 创建新会话参数
- `resume()` - 恢复会话参数

## 主要函数

### get_rollout_history()
从文件读取 rollout 历史。

**参数:**
- `path: &PathBuf` - rollout 文件路径

**返回:** `Result<InitialHistory>` - 初始历史

## 与其他模块的关系

**依赖模块:**
- `codex_protocol::protocol::{RolloutItem, InitialHistory}` - rollout 类型

**被依赖模块:**
- `codex::Session` - 持久化会话历史
- `conversation_manager` - 恢复会话时读取历史
