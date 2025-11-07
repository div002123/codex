# handlers/read_file.rs

## 文件作用

实现文件读取工具处理器，支持两种读取模式：简单切片和基于缩进的智能块读取。

## 主要结构体

### `ReadFileHandler`

文件读取工具的处理器，实现 `ToolHandler` trait。

### `ReadFileArgs`

文件读取参数：
- `file_path: String` - 文件绝对路径
- `offset: usize` - 起始行号（1-based，默认 1）
- `limit: usize` - 读取行数（默认 2000）
- `mode: ReadMode` - 读取模式（默认 Slice）
- `indentation: Option<IndentationArgs>` - 缩进模式配置

### `ReadMode`

读取模式枚举：
- `Slice` - 简单切片模式：读取连续的行范围
- `Indentation` - 缩进感知模式：智能选择代码块

### `IndentationArgs`

缩进模式配置：
- `anchor_line: Option<usize>` - 锚点行（默认为 offset）
- `max_levels: usize` - 最大缩进层级（0 = 无限制，默认 0）
- `include_siblings: bool` - 是否包含同级块（默认 false）
- `include_header: bool` - 是否包含头部注释（默认 true）
- `max_lines: Option<usize>` - 最大行数（默认为 limit）

### `LineRecord`（内部）

行记录结构：
- `number: usize` - 行号
- `raw: String` - 原始内容
- `display: String` - 显示内容（可能被截断）
- `indent: usize` - 缩进级别

## 常量

- `MAX_LINE_LENGTH: usize = 500` - 单行最大长度
- `TAB_WIDTH: usize = 4` - Tab 字符宽度
- `COMMENT_PREFIXES: &[&str]` - 注释前缀：`#`、`//`、`--`

## 主要函数

### `handle()`

处理文件读取请求：
1. 验证参数（offset、limit > 0，路径为绝对路径）
2. 根据模式调用相应的读取函数
3. 返回格式化的行列表

### Slice 模式

#### `slice::read()`

简单切片读取：
1. 打开文件
2. 逐行读取，跳过 offset 之前的行
3. 收集 limit 行数据
4. 格式化为 `L<行号>: <内容>`

### Indentation 模式

#### `indentation::read_block()`

智能代码块读取：
1. 验证参数（anchor_line、max_lines）
2. 读取整个文件到内存
3. 计算每行的有效缩进
4. 从 anchor_line 双向扩展：
   - 向上：包含父级代码块
   - 向下：包含子级代码块
5. 应用缩进和同级规则
6. 修剪空行
7. 返回格式化结果

#### `collect_file_lines()`

收集文件所有行及其缩进信息。

#### `compute_effective_indents()`

计算有效缩进（空行继承前一行的缩进）。

#### `measure_indent()`

测量行的缩进级别（Tab = 4 空格）。

## 输出格式

```
L1: fn main() {
L2:     let x = 10;
L3:     println!("{}", x);
L4: }
```

## 缩进模式示例

对于以下代码：
```rust
fn outer() {
    if cond {
        inner();
    }
    tail();
}
```

使用 `anchor_line: 3`, `max_levels: 1`：
```
L2:     if cond {
L3:         inner();
L4:     }
```

使用 `anchor_line: 3`, `max_levels: 2`：
```
L1: fn outer() {
L2:     if cond {
L3:         inner();
L4:     }
L5:     tail();
L6: }
```

## 特殊处理

### 空行处理
- 空行继承前一行的缩进级别
- 结果开头和结尾的空行会被修剪

### 注释处理
- 当 `include_header: true` 时，允许包含头部注释
- 注释不受同级限制影响

### 同级控制
- `include_siblings: false`：只包含锚点块及其父级
- `include_siblings: true`：同时包含同级块

## 错误处理

- offset 为 0：返回 "offset must be a 1-indexed line number"
- limit 为 0：返回 "limit must be greater than zero"
- 非绝对路径：返回 "file_path must be an absolute path"
- offset 超出文件：返回 "offset exceeds file length"
- anchor_line 超出文件：返回 "anchor_line exceeds file length"
- 读取失败：返回 "failed to read file"

## 与其他模块的关系

- **依赖模块**：
  - `tokio::fs` - 异步文件读取
  - `codex_utils_string::take_bytes_at_char_boundary` - 字符边界截断

- **使用场景**：
  - Slice 模式：查看文件的特定区域
  - Indentation 模式：智能提取函数、类或代码块

- **优势**：
  - 自动识别代码结构
  - 减少 token 使用（只返回相关代码）
  - 保持上下文完整性

## 测试

包含广泛的测试覆盖：
- 基础切片读取
- CRLF 处理
- 长行截断
- 缩进模式块提取
- 深度控制
- 同级包含
- 多语言示例（Python、JavaScript、C++）
- 头部注释处理
