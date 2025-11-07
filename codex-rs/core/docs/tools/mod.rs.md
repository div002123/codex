# mod.rs

## 文件作用

本文件是 tools 模块的主入口文件，负责导出子模块并提供工具调用输出格式化的核心功能。它定义了遥测预览的限制常量，并提供了将执行输出格式化为模型可理解格式的工具函数。

## 主要常量

- **TELEMETRY_PREVIEW_MAX_BYTES**: 遥测预览最大字节数（2 KiB）
- **TELEMETRY_PREVIEW_MAX_LINES**: 遥测预览最大行数（64 行）
- **TELEMETRY_PREVIEW_TRUNCATION_NOTICE**: 截断提示文本

## 主要函数

- **format_exec_output_for_model()**: 格式化执行输出为 JSON 格式，包含退出码和执行时长元数据
- **format_exec_output_str()**: 将执行输出格式化为字符串，处理超时和截断逻辑

## 导出的子模块

- `context`: 工具调用上下文
- `events`: 工具事件处理
- `handlers`: 工具处理器实现
- `orchestrator`: 工具编排器
- `parallel`: 并行执行支持
- `registry`: 工具注册表
- `router`: 工具路由器
- `runtimes`: 工具运行时
- `sandboxing`: 沙箱化支持
- `spec`: 工具规范定义

## 与其他模块的关系

- 依赖 `context_manager` 模块提供输出格式化功能
- 依赖 `exec` 模块的 `ExecToolCallOutput` 类型
- 被其他工具相关模块作为基础模块引用
