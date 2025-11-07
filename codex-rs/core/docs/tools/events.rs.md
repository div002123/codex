# events.rs

## 文件作用

本文件实现了工具执行的事件发射系统，负责在工具执行的不同阶段（开始、成功、失败）生成和发送事件消息。它提供了统一的事件处理接口，支持 Shell、ApplyPatch 和 UnifiedExec 等不同类型的工具。

## 主要结构体

- **ToolEventCtx**: 工具事件上下文，包含会话、回合、调用 ID 和差异跟踪器
- **ToolEventStage**: 工具事件阶段枚举
  - Begin: 开始阶段
  - Success: 成功阶段（带输出）
  - Failure: 失败阶段（带失败信息）
- **ToolEventFailure**: 工具事件失败类型
  - Output: 带执行输出的失败
  - Message: 带消息字符串的失败
- **ToolEmitter**: 工具事件发射器枚举
  - Shell: Shell 工具发射器
  - ApplyPatch: 补丁应用发射器
  - UnifiedExec: 统一执行发射器

## 主要方法

### ToolEventCtx
- **new()**: 创建新的事件上下文

### ToolEmitter
- **shell()**: 创建 Shell 工具发射器
- **apply_patch()**: 创建补丁应用发射器
- **unified_exec()**: 创建统一执行发射器
- **emit()**: 发射指定阶段的事件
- **begin()**: 发射开始事件
- **finish()**: 发射结束事件并返回结果

## 辅助函数

- **emit_exec_command_begin()**: 发射执行命令开始事件
- **emit_exec_end()**: 发射执行命令结束事件
- **emit_patch_end()**: 发射补丁应用结束事件

## 事件发射流程

### Shell 工具事件
1. Begin: 发送 ExecCommandBegin 事件（包含命令、工作目录、解析后的命令）
2. Success/Failure: 发送 ExecCommandEnd 事件（包含 stdout、stderr、退出码、执行时长）

### ApplyPatch 工具事件
1. Begin: 发送 PatchApplyBegin 事件，更新差异跟踪器
2. Success/Failure: 发送 PatchApplyEnd 事件，可能发送 TurnDiff 事件

### UnifiedExec 工具事件
1. Begin: 发送 ExecCommandBegin 事件
2. Success/Failure: 发送 ExecCommandEnd 事件

## 与其他模块的关系

- 使用 `Session` 的 `send_event` 方法发送事件
- 依赖 `TurnContext` 提供回合上下文
- 使用 `SharedTurnDiffTracker` 跟踪文件变更
- 与 `protocol` 模块的事件类型集成
- 支持各种工具类型的统一事件处理
- 处理 `ToolError` 并转换为 `FunctionCallError`
