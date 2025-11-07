# handlers/list_dir.rs

## 文件作用

实现目录列表工具处理器，以树状结构递归列出目录内容，支持深度控制和分页。

## 主要结构体

### `ListDirHandler`

目录列表工具的处理器，实现 `ToolHandler` trait。

### `ListDirArgs`

目录列表参数：
- `dir_path: String` - 目录路径（必须是绝对路径）
- `offset: usize` - 起始条目位置（1-based，默认 1）
- `limit: usize` - 返回条目数量（默认 25）
- `depth: usize` - 递归深度（默认 2）

### `DirEntry`（内部）

目录条目记录：
- `name: String` - 排序用的完整名称
- `display_name: String` - 显示名称
- `depth: usize` - 嵌套深度
- `kind: DirEntryKind` - 条目类型

### `DirEntryKind`（内部）

条目类型枚举：
- `Directory` - 目录（显示为 `name/`）
- `File` - 文件
- `Symlink` - 符号链接（显示为 `name@`）
- `Other` - 其他类型（显示为 `name?`）

## 常量

- `MAX_ENTRY_LENGTH: usize = 500` - 条目名称最大长度
- `INDENTATION_SPACES: usize = 2` - 每级缩进空格数

## 主要函数

### `handle()`

处理目录列表请求：
1. 验证参数（offset、limit、depth > 0，路径为绝对路径）
2. 调用 `list_dir_slice()` 获取条目
3. 格式化输出（包含绝对路径头部）

### `list_dir_slice()`

获取目录条目的指定切片：
1. 使用 BFS 收集所有条目到指定深度
2. 检查 offset 是否超出范围
3. 按字母顺序排序
4. 返回指定范围的条目
5. 如果有更多条目，添加提示信息

### `collect_entries()`

使用广度优先搜索（BFS）收集目录条目：
- 使用队列进行 BFS 遍历
- 跟踪每个条目的相对路径和深度
- 在每层内部按名称排序
- 递归收集子目录（如果深度允许）

### `format_entry_line()`

格式化单个条目为显示字符串：
- 根据深度添加缩进（每级 2 空格）
- 根据类型添加后缀（`/` 目录，`@` 符号链接，`?` 其他）

## 输出格式

```
Absolute path: /path/to/directory
file1.txt
subdir/
  nested_file.rs
  deeper/
    deep_file.md
link@
```

## 错误处理

- offset 为 0：返回 "offset must be a 1-indexed entry number"
- limit 为 0：返回 "limit must be greater than zero"
- depth 为 0：返回 "depth must be greater than zero"
- 非绝对路径：返回 "dir_path must be an absolute path"
- offset 超出范围：返回 "offset exceeds directory entry count"
- 读取失败：返回 "failed to read directory" 或 "failed to inspect entry"

## 与其他模块的关系

- **依赖模块**：
  - `tokio::fs` - 异步文件系统操作
  - `codex_utils_string` - 字符串截断工具

- **调用者**：AI 模型通过此处理器浏览目录结构

- **特点**：
  - 使用 BFS 确保同级条目完整性
  - 支持大目录的分页浏览
  - 截断过长的文件名

## 测试

包含完整的测试覆盖：
- 基础目录列表
- 符号链接处理（Unix）
- offset 超出范围
- 深度参数控制
- 大限制值处理
- 截断提示
- BFS 截断行为
