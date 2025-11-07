# context_manager/mod.rs

## 文件作用

`context_manager/mod.rs` 是上下文管理模块的入口文件，负责组织和导出对话历史管理的核心功能。

## 主要导出

### 公开导出 (pub(crate))

- **`ContextManager`**: 从 `history.rs` 导出，管理对话历史的核心结构
- **`format_output_for_model_body`**: 从 `truncate.rs` 导出，格式化输出用于模型输入

## 模块结构

该模块包含三个子模块：
- `history`: 对话历史管理核心逻辑
- `normalize`: 历史记录规范化，确保调用-输出对完整性
- `truncate`: 输出截断和格式化，控制发送给模型的内容大小

## 与其他模块的关系

- 被 `SessionState` 使用，管理会话的对话历史
- 与 `protocol` 模块交互，处理 `ResponseItem` 和协议类型
- 被 `tasks` 模块使用，获取和修改对话历史
- 提供历史记录的增删改查、规范化和截断功能
- 确保发送给模型的上下文符合令牌限制
