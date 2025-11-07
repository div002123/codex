# parse_command.rs

## 文件作用

`parse_command.rs` 解析用户输入的命令，包括 slash 命令和普通消息。

## 主要函数

### parse_command()
解析命令字符串。

**参数:**
- `input: &str` - 输入字符串

**返回:** 解析后的命令或消息

**功能:**
- 识别 slash 命令（如 `/help`、`/review`）
- 提取命令参数
- 处理普通文本消息

## 与其他模块的关系

**被依赖模块:**
- CLI - 解析用户输入
- `codex::submission_loop` - 处理命令提交
