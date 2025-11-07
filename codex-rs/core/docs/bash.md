# bash.rs

## 文件作用

使用 tree-sitter-bash 解析 Bash/Zsh 脚本，提取纯单词命令序列。

该模块支持解析由安全操作符（`&&`、`||`、`;`、`|`）连接的简单命令，拒绝包含重定向、替换、控制流等复杂结构的脚本。

## 主要函数

### try_parse_shell
```rust
pub fn try_parse_shell(shell_lc_arg: &str) -> Option<Tree>
```

使用 tree-sitter-bash 解析 shell 源代码。

**返回**：
- `Some(Tree)`：解析成功
- `None`：解析失败

### try_parse_word_only_commands_sequence
```rust
pub fn try_parse_word_only_commands_sequence(tree: &Tree, src: &str) -> Option<Vec<Vec<String>>>
```

解析只包含单词命令序列的脚本。

**返回**：
- `Some(Vec<Vec<String>>)`：所有命令都是纯单词命令
- `None`：包含不允许的结构

**允许的节点类型**：
- `program`、`list`、`pipeline`：顶层容器
- `command`、`command_name`：命令节点
- `word`、`string`、`string_content`、`raw_string`、`number`：参数节点

**允许的操作符**：
- `&&`：逻辑与
- `||`：逻辑或
- `;`：序列
- `|`：管道
- `"`、`'`：引号

**拒绝的结构**：
- 括号和子 shell：`(command)`
- 重定向：`>`, `<`, `>>`, `2>&1`
- 命令替换：`$(cmd)`, `` `cmd` ``
- 变量展开：`$VAR`, `${VAR}`
- 变量赋值：`VAR=value`
- 后台执行：`&`
- 控制流：`if`, `for`, `while`, `case`

### parse_shell_lc_plain_commands
```rust
pub fn parse_shell_lc_plain_commands(command: &[String]) -> Option<Vec<Vec<String>>>
```

解析 `bash -lc "..."` 或 `zsh -lc "..."` 调用。

**输入格式**：
```rust
["bash", "-lc", "ls -la && pwd"]
["zsh", "-lc", "echo hello"]
```

**返回**：
- `Some(Vec<Vec<String>>)`：解析成功的命令列表
- `None`：不是 `-lc` 格式或包含复杂结构

### parse_plain_command_from_node
```rust
fn parse_plain_command_from_node(cmd: tree_sitter::Node, src: &str) -> Option<Vec<String>>
```

从 `command` 节点提取单词列表。

**处理的节点类型**：
- `command_name`：提取命令名（必须是 `word`）
- `word`：直接添加
- `number`：作为单词添加
- `string`：提取双引号字符串的内容（只支持简单内容，不支持插值）
- `raw_string`：提取单引号字符串的内容

## 使用示例

### 示例 1：单个简单命令
```rust
let tree = try_parse_shell("ls -1").unwrap();
let cmds = try_parse_word_only_commands_sequence(&tree, "ls -1").unwrap();
// 结果：vec![vec!["ls", "-1"]]
```

### 示例 2：多个命令与安全操作符
```rust
let src = "ls && pwd; echo 'hi there' | wc -l";
let tree = try_parse_shell(src).unwrap();
let cmds = try_parse_word_only_commands_sequence(&tree, src).unwrap();
// 结果：vec![
//     vec!["ls"],
//     vec!["pwd"],
//     vec!["echo", "hi there"],
//     vec!["wc", "-l"],
// ]
```

### 示例 3：引号字符串
```rust
let tree = try_parse_shell("echo \"hello world\"").unwrap();
let cmds = try_parse_word_only_commands_sequence(&tree, "echo \"hello world\"").unwrap();
// 结果：vec![vec!["echo", "hello world"]]
```

### 示例 4：拒绝括号
```rust
let result = try_parse_shell("(ls)");
assert!(result.is_some()); // 解析成功但...
let cmds = try_parse_word_only_commands_sequence(&result.unwrap(), "(ls)");
assert!(cmds.is_none()); // 被拒绝
```

### 示例 5：拒绝重定向
```rust
let tree = try_parse_shell("ls > out.txt").unwrap();
let cmds = try_parse_word_only_commands_sequence(&tree, "ls > out.txt");
assert!(cmds.is_none());
```

### 示例 6：拒绝命令替换
```rust
let tree = try_parse_shell("echo $(pwd)").unwrap();
let cmds = try_parse_word_only_commands_sequence(&tree, "echo $(pwd)");
assert!(cmds.is_none());
```

### 示例 7：解析 bash -lc
```rust
let command = vec!["bash".to_string(), "-lc".to_string(), "ls && pwd".to_string()];
let parsed = parse_shell_lc_plain_commands(&command).unwrap();
// 结果：vec![vec!["ls"], vec!["pwd"]]
```

## 实现细节

### 遍历策略
使用栈（LIFO）遍历语法树：
1. 从根节点开始
2. 检查每个节点的类型
3. 如果是命名节点，检查是否在允许列表中
4. 如果是标点符号，检查是否为允许的操作符
5. 收集所有 `command` 节点
6. 按源代码位置排序（因为栈遍历会打乱顺序）
7. 解析每个命令节点

### 字符串处理
- **双引号字符串**：只支持简单的 `"string_content"` 结构，不支持嵌入的变量或命令替换
- **单引号字符串**：直接提取引号内的内容
- **转义**：不进行特殊转义处理

## 与其他模块的关系

- **shell.rs**：配合使用，提供 shell 执行的命令解析
- **exec.rs**：在某些安全检查场景中使用命令解析
- **sandboxing**：在沙箱策略决策中可能使用命令解析
- **tree-sitter-bash**：外部依赖，提供 Bash 语法解析器
