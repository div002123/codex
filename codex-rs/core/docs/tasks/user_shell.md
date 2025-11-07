# tasks/user_shell.rs

## 文件作用

实现 `UserShellCommandTask`，执行用户直接请求的 shell 命令。

## 主要结构体

### UserShellCommandTask

用户 shell 命令执行任务。

**字段：**

- `command: String` - 要执行的 shell 命令

**构造：**

- `new(command: String) -> Self` - 创建命令任务

## 常量

- `USER_SHELL_TOOL_NAME: &str = "local_shell"` - 使用的工具名称

## SessionTask 实现

### kind() -> TaskKind

返回 `TaskKind::Regular`。

### async fn run(...)

执行 shell 命令流程：

**流程：**

1. 发送 `TaskStarted` 事件
2. 根据用户的默认 shell 类型包装命令：
   - **Zsh**: `zsh -lc "command"`
   - **Bash**: `bash -lc "command"`
   - **PowerShell**: `powershell -NoProfile -Command "command"`
   - **Unknown**: 使用 `shlex::split` 解析命令
3. 构建 `ShellToolCallParams`
4. 创建 `ToolCall` 对象（使用 `local_shell` 工具）
5. 通过 `ToolCallRuntime` 执行工具调用
6. 记录错误（如果有）

**Shell 调用选项：**

- 使用登录 shell 模式 (`-l` 标志，Zsh/Bash）
- 不加载配置文件（PowerShell 使用 `-NoProfile`）
- 命令通过 `-c`/`-Command` 参数传递

**返回：**

- `Option<String>` - 始终返回 None

## 与其他模块的关系

- 实现 `SessionTask` trait
- 使用 `crate::shell::Shell` 检测用户的默认 shell
- 使用 `ToolRouter` 和 `ToolCallRuntime` 执行工具调用
- 依赖 `TurnDiffTracker` 跟踪更改
- 发送 `TaskStarted` 事件
- 使用 `ShellToolCallParams` 配置 shell 执行参数
- 允许用户直接运行 shell 命令，支持管道、重定向等 shell 特性
- 与 `tools` 模块的 shell 工具集成
