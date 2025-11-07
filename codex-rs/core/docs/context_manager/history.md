# context_manager/history.rs

## 文件作用

定义 `ContextManager` 结构体，管理对话历史的完整生命周期，包括记录、检索、规范化和令牌使用跟踪。

## 主要结构体

### ContextManager

对话历史记录管理器。

**字段：**

- `items: Vec<ResponseItem>` - 对话条目列表（从旧到新）
- `token_info: Option<TokenUsageInfo>` - 令牌使用信息

## 主要方法

### 构造和基础操作

- `new() -> Self` - 创建新的上下文管理器
- `contents(&self) -> Vec<ResponseItem>` - 返回历史记录克隆
- `replace(&mut self, items: Vec<ResponseItem>)` - 替换整个历史

### 记录管理

- `record_items<I>(&mut self, items: I)` - 记录新的对话条目
  - 过滤非 API 消息和非 ghost snapshot
  - 处理工具输出（截断）
  - 追加到历史列表

### 历史获取

- `get_history(&mut self) -> Vec<ResponseItem>` - 获取规范化后的历史
- `get_history_for_prompt(&mut self) -> Vec<ResponseItem>` - 获取用于模型提示的历史
  - 移除 ghost snapshots
  - 适合发送给 AI 模型

### 历史修改

- `remove_first_item(&mut self)` - 移除最旧的条目
  - 自动移除对应的调用-输出对

### 令牌管理

- `token_info(&self) -> Option<TokenUsageInfo>` - 获取令牌使用信息
- `update_token_info(&mut self, usage: &TokenUsage, model_context_window: Option<i64>)` - 更新令牌信息
- `set_token_usage_full(&mut self, context_window: i64)` - 标记令牌使用已满

## 内部方法

### normalize_history

规范化历史记录，确保：
1. 每个调用都有对应的输出
2. 每个输出都有对应的调用

### process_item

处理单个条目：
- 截断 `FunctionCallOutput` 和 `CustomToolCallOutput`
- 其他类型直接克隆

### remove_ghost_snapshots

从历史中移除所有 `GhostSnapshot` 条目。

## 辅助函数

### is_api_message

判断条目是否为 API 消息（非系统消息）：
- 用户/助手消息（非系统角色）
- 函数调用和输出
- 自定义工具调用和输出
- 本地 shell 调用
- 推理
- Web 搜索调用

## 与其他模块的关系

- 使用 `normalize` 模块的函数规范化历史
- 使用 `truncate` 模块的函数截断输出
- 被 `SessionState` 持有和使用
- 与 `protocol` 模块交互，处理各种 `ResponseItem` 类型
- 跟踪令牌使用，支持上下文窗口管理
- 为 `CompactTask` 提供历史记录以进行压缩
