# rollout/list.rs

## 文件作用

`rollout/list.rs` 实现 rollout 文件列表管理，用于发现和列举已保存的会话。

## 主要函数

### list_rollouts()
列出所有 rollout 文件。

**返回:** `Vec<RolloutInfo>` - rollout 信息列表

**功能:**
- 扫描 rollout 目录
- 读取元数据
- 按时间排序

## 主要结构体

### RolloutInfo
Rollout 信息。

**字段:**
- `path: PathBuf` - 文件路径
- `conversation_id: ConversationId` - 会话 ID
- `created_at: DateTime` - 创建时间
- `user_instructions: Option<String>` - 用户指令

## 与其他模块的关系

**被依赖模块:**
- CLI - 显示可恢复的会话列表
