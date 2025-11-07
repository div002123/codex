# command_safety/is_dangerous_command.rs - 危险命令检测

## 文件作用

该文件实现了危险命令的检测逻辑，识别可能导致数据丢失、系统损坏或安全风险的命令操作。

## 主要函数

### command_might_be_dangerous
- **签名**: `pub fn command_might_be_dangerous(command: &[String]) -> bool`
- **作用**: 判断命令是否可能具有危险性
- **检查流程**:
  1. 检查直接执行的危险命令
  2. 解析 `bash -lc` / `zsh -lc` 中的脚本
  3. 递归检查脚本中的每个命令

### is_dangerous_to_call_with_exec
- **签名**: `fn is_dangerous_to_call_with_exec(command: &[String]) -> bool`
- **作用**: 内部函数，检查单个命令是否危险
- **特殊处理**:
  - 递归处理 `sudo` 命令
  - 支持命令路径检查（如 `/usr/bin/git`）

## 危险命令清单

### git 危险操作
- **危险子命令**:
  - `git reset` - 重置提交历史，可能丢失代码
  - `git rm` - 删除文件并从版本控制移除
- **检测方式**: 匹配命令名 `git` 或路径结尾 `/git`

### rm 危险操作
- **危险参数**:
  - `rm -f` - 强制删除，不提示确认
  - `rm -rf` - 递归强制删除整个目录树
- **风险**: 可能导致不可恢复的数据丢失

### sudo 特殊处理
- **检测方式**: 递归检查 `sudo` 后的实际命令
- **示例**: `sudo git reset` 被识别为危险
- **意义**: 防止通过权限提升执行危险操作

## Shell 脚本支持

### bash/zsh -lc 命令
- **支持**: 解析脚本字符串，检查其中的所有命令
- **示例**:
  - `bash -lc "git reset --hard"` → 危险
  - `bash -lc "git status"` → 安全
  - `zsh -lc "git reset --hard"` → 危险

## 检测示例

### 危险命令示例
```rust
// 以下命令会被识别为危险：
command_might_be_dangerous(&["git", "reset"])           // true
command_might_be_dangerous(&["/usr/bin/git", "reset"])  // true
command_might_be_dangerous(&["rm", "-rf", "/"])         // true
command_might_be_dangerous(&["rm", "-f", "file"])       // true
command_might_be_dangerous(&["sudo", "git", "reset"])   // true
command_might_be_dangerous(&["bash", "-lc", "git reset --hard"]) // true
```

### 安全命令示例
```rust
// 以下命令不被识别为危险：
command_might_be_dangerous(&["git", "status"])          // false
command_might_be_dangerous(&["rm", "file"])             // false (没有 -f)
command_might_be_dangerous(&["ls", "-la"])              // false
```

## 设计原则

1. **黑名单机制**: 明确列出危险命令和参数组合
2. **递归检查**: 处理 sudo 和 shell 包装
3. **保守但精确**: 避免误报，同时捕获真正的危险
4. **路径无关**: 无论命令是相对路径还是绝对路径都能检测

## 安全考虑

1. **数据保护**: 防止意外删除或修改文件
2. **版本控制安全**: 保护 Git 历史不被破坏性修改
3. **权限升级检测**: 识别通过 sudo 的危险操作
4. **脚本解析**: 防止在脚本中隐藏危险命令

## 与其他模块的关系

- **依赖 `bash` 模块**: 使用 `parse_shell_lc_plain_commands()` 解析 shell 脚本
- **与 `is_safe_command` 互补**: 提供双重安全检查
- **被 `exec` 模块调用**: 在执行前评估风险
- **影响用户审批**: 危险命令需要用户明确确认
- **与沙箱系统集成**: 危险命令可能需要额外的隔离措施
