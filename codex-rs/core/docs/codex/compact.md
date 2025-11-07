# codex/compact.rs

## 文件作用

`compact.rs` 实现了会话压缩（compaction）功能，用于在对话历史过长时自动压缩旧的消息。当上下文窗口接近限制时，系统会使用AI模型总结之前的对话内容，并用摘要替换原始消息，从而保持上下文在模型限制内。

## 核心功能

### 会话压缩流程
1. 检测上下文窗口即将超限
2. 使用特殊提示让模型总结历史对话
3. 收集用户消息和模型生成的摘要
4. 构建新的压缩后历史记录
5. 保留关键信息（GhostSnapshot等）
6. 替换原历史记录

## 主要函数

### run_inline_auto_compact_task()
运行内联自动压缩任务。

**参数:**
- `sess: Arc<Session>` - 会话对象
- `turn_context: Arc<TurnContext>` - 轮次上下文

**功能:**
使用预设的压缩提示自动触发压缩流程。

### run_compact_task()
运行用户触发的压缩任务。

**参数:**
- `sess: Arc<Session>` - 会话对象
- `turn_context: Arc<TurnContext>` - 轮次上下文
- `input: Vec<UserInput>` - 用户输入

**返回:** `Option<String>` - 总是返回 None

**功能:**
- 发送 TaskStarted 事件
- 调用内部压缩逻辑
- 处理用户自定义的压缩请求

### run_compact_task_inner()
压缩任务的核心实现。

**参数:**
- `sess: Arc<Session>` - 会话对象
- `turn_context: Arc<TurnContext>` - 轮次上下文
- `input: Vec<UserInput>` - 输入内容

**功能:**
1. 克隆当前历史记录
2. 循环尝试压缩，处理上下文窗口超限
3. 如果上下文超限，移除最旧的历史项
4. 调用 `drain_to_completed()` 获取模型摘要
5. 使用 `build_compacted_history()` 构建新历史
6. 保留 GhostSnapshot 项
7. 替换会话历史
8. 持久化压缩项
9. 发送完成事件和警告

**错误处理:**
- `CodexErr::Interrupted` - 任务被中断，直接返回
- `CodexErr::ContextWindowExceeded` - 上下文窗口超限，移除最旧项后重试
- 其他错误 - 使用指数退避重试，达到最大重试次数后返回错误

### content_items_to_text()
将内容项列表转换为文本。

**参数:**
- `content: &[ContentItem]` - 内容项数组

**返回:** `Option<String>` - 合并后的文本，如果没有文本则返回 None

**功能:**
- 提取所有 InputText 和 OutputText
- 忽略图片内容
- 用换行符连接文本片段

### collect_user_messages()
从历史记录中收集用户消息。

**参数:**
- `items: &[ResponseItem]` - 响应项数组

**返回:** `Vec<String>` - 用户消息文本列表

**功能:**
- 过滤出用户消息
- 排除会话前缀消息（如AGENTS.md指令、环境上下文）
- 提取消息文本内容

### build_compacted_history()
构建压缩后的历史记录。

**参数:**
- `initial_context: Vec<ResponseItem>` - 初始上下文（系统提示等）
- `user_messages: &[String]` - 用户消息列表
- `summary_text: &str` - 模型生成的摘要

**返回:** `Vec<ResponseItem>` - 压缩后的历史记录

**功能:**
1. 保留初始上下文
2. 选择性保留最近的用户消息（最多20,000 tokens）
3. 添加压缩摘要作为用户消息
4. 如果摘要为空，使用默认文本 "(no summary available)"

### build_compacted_history_with_limit()
带大小限制的压缩历史构建。

**参数:**
- `history: Vec<ResponseItem>` - 初始历史
- `user_messages: &[String]` - 用户消息列表
- `summary_text: &str` - 摘要文本
- `max_bytes: usize` - 最大字节数限制

**返回:** `Vec<ResponseItem>` - 压缩后的历史

**功能:**
- 从最新消息开始，倒序选择用户消息
- 如果消息太长，使用 `truncate_middle()` 截断
- 确保总大小不超过限制
- 添加摘要到末尾

### drain_to_completed()
消费模型流直到完成。

**参数:**
- `sess: &Session` - 会话对象
- `turn_context: &TurnContext` - 轮次上下文
- `prompt: &Prompt` - 提示

**返回:** `CodexResult<()>` - 成功或错误

**功能:**
- 启动模型流
- 处理输出项完成事件并记录到历史
- 更新速率限制信息
- 处理完成事件并更新token使用信息
- 处理流错误

## 与其他模块的关系

**依赖模块:**
- `Session` - 会话管理
- `TurnContext` - 轮次上下文
- `client_common::ResponseEvent` - 模型响应事件
- `truncate::truncate_middle` - 文本截断工具
- `util::backoff` - 指数退避算法
- `event_mapping::parse_turn_item` - 事件解析

**被依赖模块:**
- `codex.rs` - 主模块在需要时触发压缩

**核心流程:**
上下文窗口接近限制 → 触发压缩 → 调用模型总结 → 收集用户消息 → 构建新历史 → 替换原历史

## 常量

- `SUMMARIZATION_PROMPT` - 压缩提示模板（从 `templates/compact/prompt.md` 加载）
- `COMPACT_USER_MESSAGE_MAX_TOKENS` - 压缩后保留的用户消息最大token数（20,000）

## 测试

包含单元测试验证以下功能：
- `content_items_to_text` - 文本合并逻辑
- `collect_user_messages` - 用户消息收集和过滤
- `build_compacted_history` - 压缩历史构建
- 截断过长用户消息
- 会话前缀消息过滤
