# handlers/grep_files.rs

## 文件作用

实现文件搜索工具处理器，使用 ripgrep (rg) 在文件系统中搜索匹配模式的文件。

## 主要结构体

### `GrepFilesHandler`

文件搜索工具的处理器，实现 `ToolHandler` trait。

### `GrepFilesArgs`

搜索参数结构体：
- `pattern: String` - 搜索模式（正则表达式）
- `include: Option<String>` - 文件过滤 glob 模式
- `path: Option<String>` - 搜索路径（可选，默认为当前工作目录）
- `limit: usize` - 返回结果数量限制（默认 100，最大 2000）

## 常量

- `DEFAULT_LIMIT: usize = 100` - 默认结果数量限制
- `MAX_LIMIT: usize = 2000` - 最大结果数量限制
- `COMMAND_TIMEOUT: Duration = 30s` - 命令执行超时时间

## 主要函数

### `handle()`

异步处理搜索请求的主函数：
1. 解析搜索参数
2. 验证模式和限制
3. 解析搜索路径
4. 验证路径存在性
5. 执行 ripgrep 搜索
6. 返回搜索结果

### `run_rg_search()`

执行 ripgrep 搜索的核心函数：
- 构建 `rg` 命令行参数
- 使用 `--files-with-matches` 仅返回文件路径
- 使用 `--sortr=modified` 按修改时间排序
- 支持 glob 模式过滤
- 设置 30 秒超时
- 解析输出并限制结果数量

### `verify_path_exists()`

异步验证路径是否存在。

### `parse_results()`

解析 ripgrep 输出，提取文件路径列表：
- 按换行符分割输出
- 跳过空行
- 应用结果数量限制

## 错误处理

- 空模式：返回 "pattern must not be empty"
- 零限制：返回 "limit must be greater than zero"
- 路径不存在：返回 "unable to access `<path>`"
- rg 超时：返回 "rg timed out after 30 seconds"
- rg 未安装：返回 "Ensure ripgrep is installed and on PATH"

## 返回值

成功时返回文件路径列表（每行一个路径），失败时返回 "No matches found."。

## 与其他模块的关系

- **外部依赖**：
  - `ripgrep (rg)` - 必须安装在系统上

- **调用者**：AI 模型通过此处理器搜索代码库中的文件

- **协作模块**：
  - `TurnContext` - 获取当前工作目录和解析路径
  - `ToolHandler` - 工具处理器框架

## 测试

包含完整的单元测试覆盖：
- 基础结果解析
- 结果截断
- 实际搜索功能
- glob 过滤
- 限制控制
- 无匹配处理
