# router.rs

## 文件作用

本文件实现了工具路由器，作为工具调用的中央调度中心。它负责将模型返回的不同类型的响应项（函数调用、自定义工具调用、本地 Shell 调用等）转换为统一的工具调用表示，并分发到相应的处理器执行。

## 主要结构体

- **ToolCall**: 工具调用表示，包含工具名称、调用 ID 和负载
- **ToolRouter**: 工具路由器，维护工具注册表和规范列表

## 主要方法

### ToolRouter
- **from_config()**: 从配置创建工具路由器
- **specs()**: 获取所有工具规范列表
- **tool_supports_parallel()**: 检查工具是否支持并行执行
- **build_tool_call()**: 从响应项构建工具调用
- **dispatch_tool_call()**: 分发工具调用到注册表执行
- **failure_response()**: 生成失败响应

## 核心功能

### 工具调用构建
- 支持函数调用（FunctionCall）
- 支持自定义工具调用（CustomToolCall）
- 支持本地 Shell 调用（LocalShellCall）
- 自动识别和解析 MCP 工具调用
- 特殊处理 unified_exec 工具

### 工具调用分发
- 将工具调用转换为 ToolInvocation
- 通过注册表分发到相应的处理器
- 处理成功和失败情况
- 生成标准化的响应格式

## 与其他模块的关系

- 是工具系统的核心路由层
- 依赖 `registry` 模块的 `ToolRegistry` 进行工具分发
- 依赖 `spec` 模块的 `ToolsConfig` 和 `build_specs` 进行配置
- 依赖 `context` 模块的工具调用上下文类型
- 与 `Session` 和 `TurnContext` 集成以支持工具调用的执行环境
- 使用 `codex_protocol` 定义的协议类型进行通信
