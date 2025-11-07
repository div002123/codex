# tasks/review.rs

## 文件作用

实现 `ReviewTask`，处理代码审查任务，通过子代理执行审查并解析结构化输出。

## 主要结构体

### ReviewTask

代码审查任务，启动独立的审查代理来分析代码变更。

**特点：**

- 无状态结构体（可复制、可默认构造）
- 实现 `SessionTask` trait
- 任务类型为 `TaskKind::Review`

## SessionTask 实现

### kind() -> TaskKind

返回 `TaskKind::Review`。

### async fn run(...)

执行审查任务流程：

**流程：**

1. 启动子代理对话（`start_review_conversation`）
   - 使用专门的审查提示词（`REVIEW_PROMPT`）
   - 禁用某些功能（web 搜索、图像查看）
   - 不加载项目文档
2. 处理审查事件（`process_review_events`）
   - 过滤和转发事件
   - 抑制某些内部事件
   - 解析最终审查输出
3. 退出审查模式（`exit_review_mode`）

### async fn abort(...)

任务中止时的清理：
- 调用 `exit_review_mode`，发送中断消息

## 主要函数

### start_review_conversation

启动审查子代理对话。

**配置调整：**

- 清除外部用户指令
- 设置项目文档最大字节为 0
- 禁用 web 搜索和图像查看
- 使用 `REVIEW_PROMPT` 作为基础指令

**返回：**

- `Option<async_channel::Receiver<Event>>` - 事件接收器

### process_review_events

处理审查代理发送的事件。

**行为：**

- 缓冲 agent 消息，只发送最终消息
- 抑制某些事件（ItemCompleted, AgentMessageDelta 等）
- 解析 TaskComplete 事件中的审查输出
- 处理 TurnAborted 事件

**返回：**

- `Option<ReviewOutputEvent>` - 审查输出

### parse_review_output_event

从文本解析 `ReviewOutputEvent`。

**解析策略：**

1. 尝试直接解析整个文本为 JSON
2. 提取第一个 JSON 对象并解析
3. 失败时返回包含原始文本的回退结构

### exit_review_mode

退出审查模式并记录结果。

**操作：**

1. 格式化审查发现
2. 记录开发者消息
3. 发送 `ExitedReviewMode` 事件

## 与其他模块的关系

- 实现 `SessionTask` trait
- 使用 `codex_delegate::run_codex_conversation_one_shot` 启动子代理
- 依赖 `review_format` 模块格式化审查结果
- 与 `protocol` 模块交互，发送和接收审查相关事件
- 与 `features` 模块协作，控制审查模式下的功能可用性
- 记录对话历史到会话的 `ContextManager`
