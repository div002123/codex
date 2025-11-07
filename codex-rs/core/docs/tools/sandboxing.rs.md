# sandboxing.rs

## 文件作用

本文件定义了工具沙箱化和审批流程的核心 trait 和类型，为工具执行提供安全保障机制。它整合了审批决策、沙箱偏好、工具运行时等关键组件，是工具安全执行架构的基础。

## 主要结构体

- **ApprovalStore**: 审批决策缓存，使用序列化的键存储审批决策
- **ApprovalCtx**: 审批上下文，包含会话、回合、调用 ID、重试原因和风险评估
- **ToolCtx**: 工具上下文，包含会话、回合、调用 ID 和工具名称
- **SandboxRetryData**: 沙箱重试数据，包含命令和工作目录
- **SandboxAttempt**: 沙箱尝试，包含沙箱类型、策略、管理器和相关路径
- **ToolError**: 工具错误枚举（Rejected 或 Codex 错误）

## 主要枚举

- **SandboxablePreference**: 沙箱偏好
  - Auto: 自动选择
  - Require: 要求沙箱
  - Forbid: 禁止沙箱

## 主要 Trait

### Approvable<Req>
工具审批能力 trait，定义了：
- **approval_key()**: 生成审批缓存键
- **wants_escalated_first_attempt()**: 是否需要首次提升权限
- **should_bypass_approval()**: 是否应绕过审批
- **wants_initial_approval()**: 是否需要初始审批
- **wants_no_sandbox_approval()**: 是否需要无沙箱审批
- **start_approval_async()**: 异步启动审批流程

### Sandboxable
沙箱化能力 trait，定义了：
- **sandbox_preference()**: 返回沙箱偏好
- **escalate_on_failure()**: 失败时是否升级权限

### ToolRuntime<Req, Out>
工具运行时 trait，组合了 Approvable 和 Sandboxable，定义了：
- **run()**: 异步执行工具

### ProvidesSandboxRetryData
提供沙箱重试数据的 trait：
- **sandbox_retry_data()**: 返回重试所需的命令和工作目录

## 主要函数

- **with_cached_approval()**: 带缓存的审批决策获取，支持会话级别的审批缓存

## 主要方法

### ApprovalStore
- **get()**: 从缓存获取审批决策
- **put()**: 向缓存存储审批决策

### SandboxAttempt
- **env_for()**: 根据命令规范生成执行环境

## 与其他模块的关系

- 是工具安全执行的核心抽象层
- 被 `orchestrator` 模块使用以实现工具编排
- 被具体的工具运行时（如 handlers/runtimes）实现
- 依赖 `sandboxing` 模块的 `SandboxManager` 和相关类型
- 使用 `Session` 进行风险评估和审批请求
- 与 `error` 模块集成处理各种错误类型
- 支持会话级别的审批决策缓存
