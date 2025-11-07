# openai_model_info.rs

## 文件作用

`openai_model_info.rs` 提供了OpenAI模型的元数据信息查询功能。该文件维护了一个模型信息数据库，包含各个模型的上下文窗口大小、最大输出token数、自动压缩阈值等关键参数。这些信息用于客户端进行智能的上下文管理和token计数。

文件支持主要的OpenAI模型系列（GPT-3.5、GPT-4、GPT-4o、GPT-5、o3、o4等），以及开源模型和实验性模型，为每个模型提供了准确的技术规格。

## 主要结构体列表

- `ModelInfo` - 模型元数据结构
  - `context_window` - 上下文窗口大小（token数）
  - `max_output_tokens` - 最大输出token数
  - `auto_compact_token_limit` - 自动压缩token阈值（可选）

## 主要函数/方法列表

### ModelInfo 方法：
- `new()` - 创建新的ModelInfo实例（常量函数）
- `default_auto_compact_limit()` - 计算默认的自动压缩限制（上下文窗口的90%）

### 模块函数：
- `get_model_info()` - 根据模型家族获取模型信息
  - 支持的模型：
    - gpt-oss-20b / gpt-oss-120b（96K上下文，32K输出）
    - o3（200K上下文，100K输出）
    - o4-mini（200K上下文，100K输出）
    - codex-mini-latest（200K上下文，100K输出）
    - gpt-4.1 系列（1047K上下文，32K输出）
    - gpt-4o 系列（128K上下文，4K-16K输出，取决于版本）
    - gpt-3.5-turbo（16K上下文，4K输出）
    - gpt-5 系列（272K上下文，128K输出）
    - gpt-5-codex（272K上下文，128K输出）
    - codex- 前缀（272K上下文，128K输出）

### 常量：
- `CONTEXT_WINDOW_272K` - 272,000 token 上下文窗口
- `MAX_OUTPUT_TOKENS_128K` - 128,000 token 最大输出

## 与其他模块的关系

- **被 client 使用**: `client.rs` 调用 `get_model_info()` 来获取模型的上下文窗口和自动压缩限制
- **依赖 model_family**: 接受 `ModelFamily` 作为参数，使用其slug字段进行模型匹配
- **配合 config**: 提供的信息可以被用户配置覆盖（通过 config.toml）
- **上下文管理**: 提供的 `auto_compact_token_limit` 用于决定何时触发对话历史压缩
- **token计数**: 提供的窗口大小信息用于智能管理对话上下文
- **模型版本追踪**: 文档中包含了模型版本和链接，便于更新
- **开源模型支持**: 为OSS模型提供合理的token分配策略（3/4输入，1/4输出）
- **内部使用**: 标记为 `pub(crate)`，仅在crate内部使用
- **可选返回**: 返回 `Option<ModelInfo>`，允许未知模型返回None
- **常量优化**: 使用const函数支持编译时计算
