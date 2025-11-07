# custom_prompts.rs

## 文件作用

`custom_prompts.rs` 实现自定义提示（slash命令）的发现和解析功能，从文件系统加载Markdown格式的提示模板。

## 主要函数

### default_prompts_dir()
返回默认提示目录路径（`$CODEX_HOME/prompts`）。

### discover_prompts_in()
发现指定目录中的所有提示文件。

**功能:**
- 扫描目录中的 `.md` 文件
- 解析 YAML frontmatter（description、argument-hint）
- 返回排序后的 `CustomPrompt` 列表

### discover_prompts_in_excluding()
发现提示文件，排除指定名称。

### parse_frontmatter()
解析Markdown文件的YAML frontmatter。

**支持的键:**
- `description` - 提示描述
- `argument-hint` / `argument_hint` - 参数提示

## 与其他模块的关系

**依赖模块:**
- `codex_protocol::custom_prompts::CustomPrompt` - 提示结构定义

**被依赖模块:**
- CLI/UI - 显示可用的自定义提示
- 命令解析 - 执行用户选择的提示
