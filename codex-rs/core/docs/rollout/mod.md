# rollout/mod.rs

## 文件作用

`rollout/mod.rs` 是 rollout 模块的入口，导出子模块。

## 子模块

### list
Rollout 列表管理。

### policy
Rollout 策略定义。

### recorder
Rollout 记录器，负责持久化会话历史。

## 与其他模块的关系

**被依赖模块:**
- `codex::Session` - 使用 recorder 持久化历史
- `conversation_manager` - 从 rollout 恢复会话
