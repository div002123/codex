# message_history.rs

## 文件作用

`message_history.rs` 实现会话历史管理，负责记录、检索和操作对话历史。

## 主要结构体

### MessageHistory
消息历史记录。

**方法:**
- `new()` - 创建新历史
- `record_items()` - 记录响应项
- `get_history()` - 获取完整历史
- `get_history_for_prompt()` - 获取用于提示的历史
- `remove_first_item()` - 移除第一项（压缩时使用）

## 主要函数

### history_metadata()
获取历史元数据。

**功能:**
加载和解析历史相关的元信息。

## 与其他模块的关系

**依赖模块:**
- `codex_protocol::models::ResponseItem` - 历史项类型

**被依赖模块:**
- `codex::Session` - 管理会话历史
- `compact` - 压缩历史时操作
