# command_safety/windows_safe_commands.rs - Windows 安全命令检测

## 文件作用

该文件实现了 Windows 平台专用的安全命令检测逻辑，主要针对 PowerShell 命令进行严格的安全性验证。

## 主要函数

### is_safe_command_windows
- **签名**: `pub fn is_safe_command_windows(command: &[String]) -> bool`
- **作用**: 判断 Windows 命令是否安全
- **策略**: 仅允许明确的只读 PowerShell 调用
- **返回**: 非 PowerShell 命令一律返回 `false`

### try_parse_powershell_command_sequence
- **作用**: 解析 PowerShell 调用，提取命令序列
- **返回**: `Option<Vec<Vec<String>>>` - 命令列表
- **格式**: 每个 Vec 代表管道中的一个命令

### parse_powershell_invocation
- **作用**: 解析 PowerShell 参数并提取脚本内容
- **支持格式**:
  - `powershell -Command "script"`
  - `pwsh -c "script"`
  - `powershell -Command:script`
- **过滤**: 拒绝危险参数如 `-EncodedCommand`, `-File`, `-ExecutionPolicy`

### parse_powershell_script
- **作用**: 将 PowerShell 脚本字符串分词并分割为命令
- **使用**: shlex 分词器进行词法分析
- **返回**: 命令序列列表

### split_into_commands
- **作用**: 按管道和分隔符分割命令
- **支持分隔符**: `|`, `||`, `&&`, `;`
- **安全检查**: 拒绝包含重定向符号或命令替换的 token

## PowerShell 可执行文件

### is_powershell_executable
- **识别的可执行文件**:
  - `powershell`
  - `powershell.exe`
  - `pwsh`
  - `pwsh.exe`
- **大小写**: 不敏感匹配

## PowerShell 安全命令白名单

### is_safe_powershell_command
- **作用**: 验证单个 PowerShell 命令是否在安全白名单中

### 只读 Cmdlet
- `echo`, `write-output`, `write-host` - 输出命令
- `dir`, `ls`, `get-childitem`, `gci` - 列出文件
- `cat`, `type`, `gc`, `get-content` - 读取内容
- `select-string`, `sls`, `findstr` - 文本搜索
- `measure-object`, `measure` - 统计计数
- `get-location`, `gl`, `pwd` - 当前位置
- `test-path`, `tp` - 测试路径存在
- `resolve-path`, `rvpa` - 解析路径
- `select-object`, `select` - 选择对象
- `get-item` - 获取项信息

### 特殊命令处理

#### git 命令
- **委托**: `is_safe_git_command()`
- **安全子命令**: `status`, `log`, `show`, `diff`, `cat-file`
- **支持参数**: `-c`, `--config`, `--git-dir`, `--work-tree`

#### rg 命令 (ripgrep)
- **委托**: `is_safe_ripgrep()`
- **危险参数**: `--pre`, `--hostname-bin`, `--search-zip`, `-z`

### 明确禁止的 Cmdlet
- `set-content` - 写入文件
- `add-content` - 追加内容
- `out-file` - 输出到文件
- `new-item` - 创建项
- `remove-item` - 删除项
- `move-item` - 移动项
- `copy-item` - 复制项
- `rename-item` - 重命名项
- `start-process` - 启动进程
- `stop-process` - 停止进程

## 安全验证层级

### 1. 参数过滤
- 拒绝 `-EncodedCommand` (可能混淆恶意代码)
- 拒绝 `-File` (执行脚本文件)
- 拒绝 `-ExecutionPolicy` (修改安全策略)
- 拒绝 `-WindowStyle` (隐藏窗口)
- 允许 `-NoLogo`, `-NoProfile`, `-NonInteractive` (无害开关)

### 2. 脚本解析
- 使用 shlex 进行词法分析
- 检测并拒绝重定向符号 (`>`, `>>`, `<`, `2>&1` 等)
- 检测并拒绝命令替换 (`$(...)`, `` `...` ``)
- 检测并拒绝调用操作符 (`&`)

### 3. 命令验证
- 检查每个命令是否在白名单中
- 检查嵌套命令（括号内的命令）
- 递归验证管道中的所有命令

## 安全示例

### 允许的命令
```powershell
powershell -Command "Get-ChildItem -Path ."
pwsh -NoProfile -Command "git status"
powershell Get-Content Cargo.toml
pwsh -Command "rg pattern | Measure-Object"
```

### 拒绝的命令
```powershell
powershell -Command "Remove-Item foo.txt"        # 删除文件
powershell -Command "echo hi > out.txt"          # 重定向
powershell -Command "& Remove-Item foo"          # 调用操作符
powershell -EncodedCommand <base64>              # 编码命令
powershell -Command "rg --pre cat pattern"       # 危险参数
```

## 设计原则

1. **白名单机制**: 仅明确安全的 cmdlet 被允许
2. **深度验证**: 多层检查确保无遗漏
3. **保守策略**: 任何不确定的命令均被拒绝
4. **平台特定**: 针对 Windows/PowerShell 的特殊语法
5. **别名支持**: 识别常用 PowerShell 别名（如 `ls`、`cat`）

## 与其他模块的关系

- **被 `is_safe_command.rs` 调用**: 在 Windows 平台上提供命令检查
- **依赖 `shlex`**: 用于 PowerShell 脚本的词法分析
- **与 `exec` 模块集成**: 决定命令是否需要用户审批
- **Windows 专有**: 仅在 `#[cfg(target_os = "windows")]` 下编译
