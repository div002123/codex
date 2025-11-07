# model_family.rs

## 文件作用

`model_family.rs` 定义了模型家族的概念和配置。模型家族将具有相似特性的模型分组，并为每个家族配置特定的行为参数，如是否支持推理摘要、是否需要特殊的apply_patch指令、使用哪个工具API等。该文件通过模型slug（如"gpt-5"、"gpt-4.1"）识别模型并返回相应的配置。

文件使用宏 `model_family!` 来简化模型家族配置的定义，为不同的模型系列（GPT-3.5、GPT-4.x、GPT-5、o3、codex等）提供了默认值和覆盖配置。

## 主要结构体列表

- `ModelFamily` - 模型家族配置结构
  - `slug` - 完整的模型slug
  - `family` - 模型家族名称
  - `needs_special_apply_patch_instructions` - 是否需要特殊的apply_patch指令
  - `supports_reasoning_summaries` - 是否支持推理摘要
  - `reasoning_summary_format` - 推理摘要格式
  - `uses_local_shell_tool` - 是否使用local_shell工具
  - `supports_parallel_tool_calls` - 是否支持并行工具调用
  - `apply_patch_tool_type` - apply_patch工具类型（可选）
  - `base_instructions` - 基础指令文本
  - `experimental_supported_tools` - 实验性支持的工具列表
  - `effective_context_window_percent` - 有效上下文窗口百分比
  - `support_verbosity` - 是否支持verbosity设置

## 主要函数/方法列表

- `find_family_for_model()` - 根据模型slug查找对应的模型家族配置
  - 支持的模型家族：
    - o3 系列
    - o4-mini 系列
    - codex-mini-latest
    - gpt-4.1 系列
    - gpt-4o 系列
    - gpt-3.5 系列
    - gpt-5 系列（包括gpt-5-codex）
    - gpt-oss 开源模型
    - test-gpt-5-codex（测试模型）
    - codex-exp-（实验性模型）
    - codex-（生产模型）

- `derive_default_model_family()` - 为未知模型派生默认的模型家族配置

### 宏：
- `model_family!` - 用于简化模型家族配置定义的宏，提供默认值并允许覆盖特定字段

## 与其他模块的关系

- **被 client 使用**: `client.rs` 使用模型家族信息来确定如何与模型通信
- **被 client_common 使用**: `client_common.rs` 使用模型家族来生成正确的指令和参数
- **依赖 config**: 使用 `config::types::ReasoningSummaryFormat`
- **依赖 tools**: 使用 `tools::handlers::apply_patch::ApplyPatchToolType`
- **关联 openai_model_info**: 模型家族名称应该能够与 `openai_model_info::get_model_info` 配合使用
- **配置中心**: 作为模型行为配置的中央定义点
- **指令管理**: 引用prompt.md和gpt_5_codex_prompt.md作为基础指令
- **特性标志**: 通过布尔字段控制不同模型的特性启用/禁用
- **扩展性**: 通过实验性工具列表支持模型特定的beta功能
- **默认值**: 为所有未知模型提供合理的默认配置
- **百分比控制**: 通过 `effective_context_window_percent` 控制上下文窗口的实际可用空间
