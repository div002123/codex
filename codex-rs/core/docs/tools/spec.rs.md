# spec.rs

## 文件作用

本文件定义了工具规范（Tool Specification）的核心结构和配置逻辑。它负责根据模型家族和特性标志生成适配不同 LLM 的工具定义，包括 JSON Schema 定义、工具创建函数以及 MCP 工具转换逻辑。

## 主要结构体

- **ConfigShellToolType**: Shell 工具类型枚举（Default、Local、UnifiedExec）
- **ToolsConfig**: 工具配置，包含 shell 类型、补丁工具类型、web 搜索等配置
- **ToolsConfigParams**: 工具配置参数，包含模型家族和特性标志
- **JsonSchema**: 通用 JSON Schema 子集定义，支持 Boolean、String、Number、Array、Object 类型
- **AdditionalProperties**: 额外属性配置（布尔值或 Schema）
- **ApplyPatchToolArgs**: Apply Patch 工具参数

## 主要函数

- **create_exec_command_tool()**: 创建统一执行命令工具规范
- **create_write_stdin_tool()**: 创建写入标准输入工具规范
- **create_shell_tool()**: 创建 Shell 工具规范
- **create_view_image_tool()**: 创建图像查看工具规范
- **create_test_sync_tool()**: 创建测试同步工具规范
- **create_grep_files_tool()**: 创建文件搜索工具规范
- **create_read_file_tool()**: 创建文件读取工具规范
- **create_list_dir_tool()**: 创建目录列表工具规范
- **create_list_mcp_resources_tool()**: 创建 MCP 资源列表工具规范
- **create_read_mcp_resource_tool()**: 创建读取 MCP 资源工具规范
- **build_specs()**: 根据配置构建工具注册表
- **create_tools_json_for_responses_api()**: 为 Responses API 创建工具 JSON
- **create_tools_json_for_chat_completions_api()**: 为 Chat Completions API 创建工具 JSON
- **mcp_tool_to_openai_tool()**: 将 MCP 工具转换为 OpenAI 工具格式
- **sanitize_json_schema()**: 清理和标准化 JSON Schema

## 与其他模块的关系

- 被 `router` 模块使用以创建工具路由器
- 依赖 `registry` 模块的 `ToolRegistryBuilder`
- 依赖 `handlers` 模块的各种工具处理器
- 与 `features` 和 `model_family` 模块集成以支持特性标志和模型特定配置
- 支持 MCP（Model Context Protocol）工具的转换和集成
