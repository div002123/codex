# lib.rs

## 文件作用

`lib.rs` 是 `codex-core` 库的根模块文件。它作为整个核心库的入口点，负责声明和导出所有子模块、类型和公共接口。该文件通过模块系统组织代码结构，并通过 `pub use` 语句选择性地重新导出关键类型和函数，为库的使用者提供清晰的API接口。

此文件还配置了重要的 lint 规则，禁止直接向标准输出/错误流写入（`clippy::print_stdout`, `clippy::print_stderr`），以确保库代码中所有用户可见的输出都通过适当的抽象层（如TUI或tracing栈）进行。

## 主要模块列表

- `auth` - 认证管理模块
- `bash` - Bash shell 集成
- `client` - 模型客户端（私有）
- `client_common` - 客户端通用功能（私有）
- `codex` - Codex 核心功能
- `config` - 配置管理
- `config_loader` - 配置加载器
- `error` - 错误处理
- `exec` - 命令执行
- `exec_env` - 执行环境
- `features` - 功能标志
- `git_info` - Git 信息
- `mcp` - MCP协议支持
- `model_provider_info` - 模型提供者信息（私有）
- `model_family` - 模型家族
- `token_data` - Token 数据
- `sandboxing` - 沙箱功能
- `shell` - Shell 集成
- `spawn` - 进程生成
- `terminal` - 终端交互
- 其他多个内部模块（apply_patch, chat_completions, conversation_manager等）

## 主要导出类型

### 从子模块重新导出的类型：
- `ModelClient` - 模型客户端
- `ModelProviderInfo` - 模型提供者信息
- `WireApi` - 线协议API类型
- `Prompt` - 提示词结构
- `ResponseEvent` - 响应事件
- `ResponseStream` - 响应流
- `CodexConversation` - Codex 对话
- `ConversationManager` - 对话管理器
- `AuthManager` - 认证管理器
- `CodexAuth` - Codex 认证
- `RolloutRecorder` - 会话记录器
- `ConversationItem`, `ConversationsPage`, `Cursor` - 对话列表相关类型

### 从 codex-protocol 重新导出：
- `protocol` - 协议类型（完整命名空间）
- `protocol_config_types` - 协议配置类型
- `ContentItem`, `ResponseItem` - 内容和响应项
- `LocalShellAction`, `LocalShellExecAction`, `LocalShellStatus` - Shell 操作相关

## 主要函数/方法列表

本文件主要通过重新导出提供API：
- `built_in_model_providers()` - 获取内置模型提供者列表
- `create_oss_provider_with_base_url()` - 创建开源模型提供者
- `is_safe_command()` - 检查命令是否安全
- `get_platform_sandbox()` - 获取平台沙箱
- `set_windows_sandbox_enabled()` - 设置Windows沙箱状态
- `content_items_to_text()` - 将内容项转换为文本
- `parse_turn_item()` - 解析轮次项
- `find_conversation_path_by_id_str()` - 通过ID查找对话路径
- `parse_cursor()` - 解析游标
- `read_head_for_summary()` - 读取摘要头部

## 与其他模块的关系

- **入口模块**: 作为库的根模块，所有其他模块都通过它进行组织
- **API网关**: 选择性地重新导出内部模块的公共API，隐藏实现细节
- **协议桥接**: 重新导出 `codex-protocol` crate 的类型，确保现有的 `codex_core::protocol::...` 引用继续工作
- **模块组织**: 通过 `pub mod` 和私有 `mod` 声明控制模块的可见性
- **常量定义**: 定义了一些全局常量如 `CODEX_APPLY_PATCH_ARG1`、`REVIEW_PROMPT` 等
- **依赖中心**: 协调各个功能模块（认证、配置、执行、沙箱等）的集成
