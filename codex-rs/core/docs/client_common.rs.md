# client_common.rs

## 文件作用

`client_common.rs` 定义了客户端通信的通用数据结构和类型。该文件包含了与AI模型交互的核心数据结构，如提示词（Prompt）、响应事件（ResponseEvent）、工具规范（ToolSpec）等。它还处理了不同模型家族对工具调用输出格式的特殊需求，特别是针对 Freeform 工具（如 apply_patch）的输出序列化逻辑。

文件提供了构建Responses API请求payload的结构，并处理模型特定的指令和参数（如推理能力、verbosity等）。

## 主要结构体列表

- `Prompt` - 表示一次模型调用的完整输入（上下文、工具、指令等）
- `ResponseEvent` - 表示从模型返回的各种事件类型（枚举）
- `Reasoning` - 推理参数配置
- `TextFormatType` - 文本格式类型（枚举）
- `TextFormat` - 文本格式规范
- `TextControls` - GPT-5的文本控制参数
- `OpenAiVerbosity` - OpenAI verbosity级别（枚举）
- `ResponsesApiRequest` - Responses API请求的序列化结构
- `ResponseStream` - 响应事件流包装器
- `ToolSpec` - 工具规范（枚举）
  - Function - 标准函数工具
  - LocalShell - 本地Shell工具
  - WebSearch - Web搜索工具
  - Freeform - 自由格式工具
- `ResponsesApiTool` - Responses API的函数工具
- `FreeformTool` - 自由格式工具
- `FreeformToolFormat` - 自由格式工具的格式定义

## 主要函数/方法列表

### Prompt 方法：
- `get_full_instructions()` - 获取完整的指令文本（包括基础指令和特殊工具指令）
- `get_formatted_input()` - 获取格式化的输入（处理Shell输出的重新序列化）

### 辅助函数：
- `create_reasoning_param_for_request()` - 为请求创建推理参数
- `create_text_param_for_request()` - 为请求创建文本控制参数
- `reserialize_shell_outputs()` - 重新序列化Shell输出（针对Freeform工具）
- `is_shell_tool_name()` - 检查是否为Shell工具名称
- `parse_structured_shell_output()` - 解析结构化Shell输出
- `build_structured_output()` - 构建结构化输出字符串
- `extract_total_output_lines()` - 提取总输出行数
- `strip_total_output_header()` - 移除总输出头部

### ToolSpec 方法：
- `name()` - 获取工具名称

### ResponseStream 实现：
- `poll_next()` - 实现 Stream trait 的轮询方法

## 与其他模块的关系

- **被 client 使用**: `client.rs` 使用这里定义的 `Prompt`、`ResponseStream`、`ResponseEvent` 等类型
- **依赖 model_family**: 使用 `ModelFamily` 来确定模型特定的行为（如是否需要特殊指令）
- **依赖 tools::spec**: 使用 `JsonSchema` 和其他工具规范类型
- **使用 codex-protocol**: 使用 `ResponseItem`、`config_types` 中的配置枚举
- **使用 codex-apply-patch**: 引用 `APPLY_PATCH_TOOL_INSTRUCTIONS` 常量
- **被多个模块使用**: 作为客户端通信的数据契约，被多个高层模块使用
- **定义常量**: 导出 `REVIEW_PROMPT`、`REVIEW_EXIT_SUCCESS_TMPL`、`REVIEW_EXIT_INTERRUPTED_TMPL` 等常量
- **序列化支持**: 所有结构体都实现了 serde 的 Serialize/Deserialize trait，用于JSON序列化
