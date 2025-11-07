# parallel.rs

## 文件作用

本文件实现了工具调用的并行执行运行时，负责管理工具调用的并发执行、取消处理和输出格式化。它使用读写锁机制来控制工具的并行执行，确保不支持并行的工具顺序执行，而支持并行的工具可以并发运行。

## 主要结构体

- **ToolCallRuntime**: 工具调用运行时，管理并行工具调用的执行

## 主要方法

### ToolCallRuntime
- **new()**: 创建新的工具调用运行时实例
- **handle_tool_call()**: 处理工具调用，返回一个 Future
  - 检查工具是否支持并行执行
  - 管理并行执行锁（读锁用于并行工具，写锁用于顺序工具）
  - 等待工具门控（tool gate）就绪
  - 支持取消令牌（cancellation token）
  - 处理任务中止和错误情况

### 辅助方法
- **aborted_response()**: 生成中止响应
- **abort_message()**: 生成中止消息，针对不同工具类型定制消息格式

## 核心功能

### 并行执行控制
- 使用 `Arc<RwLock<()>>` 实现并行控制
- 支持并行的工具获取读锁，可以并发执行
- 不支持并行的工具获取写锁，独占执行

### 取消处理
- 支持 CancellationToken 中止工具执行
- 记录执行时长
- 生成针对不同工具类型的中止消息

### 就绪门控
- 使用 `tool_call_gate` 实现执行前的就绪等待
- 确保工具在适当时机执行

### 任务管理
- 使用 `AbortOnDropHandle` 确保任务在 drop 时自动中止
- 使用 `tokio::select!` 处理取消和正常执行的竞争

## 与其他模块的关系

- 依赖 `router` 模块的 `ToolRouter` 和 `ToolCall` 进行工具分发
- 使用 `context` 模块的 `SharedTurnDiffTracker` 跟踪变更
- 集成 `Session` 和 `TurnContext` 提供执行环境
- 使用 `Readiness` 机制实现就绪门控
- 处理各种错误类型（`FunctionCallError`、`CodexErr`）
