# handlers/mod.rs

## 文件作用

handlers 模块的入口文件，负责组织和导出所有工具处理器（Tool Handler）实现。

## 主要内容

### 模块声明

- `apply_patch` - 应用补丁处理器模块（公开）
- `grep_files` - 文件搜索处理器模块
- `list_dir` - 目录列表处理器模块
- `mcp` - MCP 工具调用处理器模块
- `mcp_resource` - MCP 资源处理器模块
- `plan` - 计划工具处理器模块
- `read_file` - 读取文件处理器模块
- `shell` - Shell 命令处理器模块
- `test_sync` - 测试同步处理器模块
- `unified_exec` - 统一执行处理器模块
- `view_image` - 查看图像处理器模块

### 公开导出

导出所有处理器的具体实现类型：

- `ApplyPatchHandler` - 应用补丁处理器
- `GrepFilesHandler` - 文件搜索处理器
- `ListDirHandler` - 目录列表处理器
- `McpHandler` - MCP 工具调用处理器
- `McpResourceHandler` - MCP 资源处理器
- `PlanHandler` - 计划工具处理器
- `ReadFileHandler` - 读取文件处理器
- `ShellHandler` - Shell 命令处理器
- `TestSyncHandler` - 测试同步处理器
- `UnifiedExecHandler` - 统一执行处理器
- `ViewImageHandler` - 查看图像处理器

同时导出：
- `PLAN_TOOL` - 计划工具的规范定义

## 与其他模块的关系

- **上游依赖**：被 `tools` 模块使用，作为工具处理器的集合
- **下游依赖**：包含所有具体的处理器实现模块
- **协作模块**：与 `runtimes` 模块配合，handlers 负责处理工具调用，runtimes 负责实际执行
