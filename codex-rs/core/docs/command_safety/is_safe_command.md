# command_safety/is_safe_command.rs - 安全命令检测

## 文件作用

该文件实现了已知安全命令的检测逻辑，维护了一个可自动批准执行的命令白名单，并对特定命令的参数进行安全性验证。

## 主要函数

### is_known_safe_command
- **签名**: `pub fn is_known_safe_command(command: &[String]) -> bool`
- **作用**: 判断命令是否为已知安全命令
- **检查流程**:
  1. 规范化命令（如 zsh 转为 bash）
  2. 检查 Windows 平台安全命令（如果适用）
  3. 检查直接调用的安全命令
  4. 解析 `bash -lc` 组合命令并逐个验证

### is_safe_to_call_with_exec
- **签名**: `fn is_safe_to_call_with_exec(command: &[String]) -> bool`
- **作用**: 内部函数，检查单个命令是否可安全执行
- **检查内容**: 基于命令名称和参数的白名单匹配

## 安全命令白名单

### 基础命令（无条件安全）
- `cat` - 读取文件内容
- `cd` - 切换目录
- `echo` - 输出文本
- `false` - 返回失败状态
- `grep` - 文本搜索
- `head` - 显示文件开头
- `ls` - 列出文件
- `nl` - 带行号显示
- `pwd` - 显示当前目录
- `tail` - 显示文件末尾
- `true` - 返回成功状态
- `wc` - 统计字数
- `which` - 查找命令路径

### 条件安全命令

#### find
- **安全条件**: 不包含以下危险选项
  - `-exec`, `-execdir`, `-ok`, `-okdir` - 执行任意命令
  - `-delete` - 删除文件
  - `-fls`, `-fprint`, `-fprint0`, `-fprintf` - 写入文件

#### rg (ripgrep)
- **安全条件**: 不包含以下选项
  - `--pre` - 执行预处理命令
  - `--hostname-bin` - 执行主机名命令
  - `--search-zip`, `-z` - 搜索压缩文件（调用外部工具）

#### git
- **安全子命令**: `branch`, `status`, `log`, `diff`, `show`
- **限制**: 仅这些只读操作被允许

#### cargo
- **安全子命令**: `check`
- **限制**: 仅检查编译，不执行代码

#### sed
- **安全模式**: `sed -n {N|M,N}p`
- **限制**: 仅允许打印特定行的只读操作

## Shell 脚本支持

### bash -lc 命令序列
- **支持格式**: `bash -lc "command1 && command2 || command3"`
- **安全运算符**: `&&`, `||`, `;`, `|`
- **验证方式**: 解析脚本，验证每个命令均为安全命令
- **限制**: 不支持重定向、子 shell、变量等复杂语法

## 辅助函数

### is_valid_sed_n_arg
- **作用**: 验证 sed -n 的行号参数格式
- **支持格式**:
  - 单行: `{N}p` (如 `5p`)
  - 范围: `{M,N}p` (如 `1,5p`)

## 设计原则

1. **白名单机制**: 仅明确列出的命令被认为安全
2. **参数验证**: 对可能危险的命令参数进行细致检查
3. **保守策略**: 有疑问时拒绝，而非批准
4. **组合命令支持**: 支持安全命令的逻辑组合

## 与其他模块的关系

- **依赖 `bash` 模块**: 使用 `parse_shell_lc_plain_commands()` 解析 shell 命令
- **被 `exec` 模块调用**: 决定是否可以自动执行命令
- **与 `is_dangerous_command` 协作**: 共同构建完整的安全检查体系
- **在 Windows 上依赖 `windows_safe_commands`**: 处理平台特定命令
