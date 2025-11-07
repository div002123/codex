# registry.rs

## 文件作用

本文件实现了工具注册表系统，负责管理工具处理器的注册、查找和分发。它提供了工具调用的核心分发机制，将工具调用路由到相应的处理器并处理结果。

## 主要结构体

- **ToolKind**: 工具类型枚举（Function、Mcp）
- **ToolHandler**: 工具处理器 trait，定义了工具处理的核心接口
- **ToolRegistry**: 工具注册表，维护工具名称到处理器的映射
- **ConfiguredToolSpec**: 配置后的工具规范，包含工具规范和并行支持标志
- **ToolRegistryBuilder**: 工具注册表构建器，用于构建工具注册表

## 主要 Trait

- **ToolHandler**:
  - `kind()`: 返回工具类型
  - `matches_kind()`: 检查工具类型是否匹配负载
  - `handle()`: 异步处理工具调用

## 主要方法

### ToolRegistry
- **new()**: 创建新的工具注册表
- **handler()**: 根据工具名称获取处理器
- **dispatch()**: 分发工具调用到相应的处理器

### ToolRegistryBuilder
- **new()**: 创建新的构建器
- **push_spec()**: 添加工具规范
- **push_spec_with_parallel_support()**: 添加带并行支持的工具规范
- **register_handler()**: 注册工具处理器
- **build()**: 构建最终的注册表和规范列表

## 主要函数

- **unsupported_tool_call_message()**: 生成不支持的工具调用错误消息

## 与其他模块的关系

- 被 `router` 模块使用以管理工具调用分发
- 依赖 `context` 模块的 `ToolInvocation` 和 `ToolOutput`
- 与 `spec` 模块协作构建工具配置
- 使用 `function_tool` 模块的 `FunctionCallError` 处理错误
- 集成 OTEL（OpenTelemetry）事件管理器进行遥测记录
