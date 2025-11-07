# handlers/unified_exec.rs

## 文件作用

实现统一执行（Unified Exec）工具处理器，提供持久化的交互式 Shell 会话管理，支持命令执行和标准输入写入。

## 主要结构体

### `UnifiedExecHandler`

统一执行工具的处理器，实现 `ToolHandler` trait。

**支持的工具**：
- `exec_command` - 执行新命令（创建或使用现有会话）
- `write_stdin` - 向现有会话的标准输入写入数据

### `ExecCommandArgs`

执行命令参数：
- `cmd: String` - 要执行的命令
- `shell: String` - Shell 路径（默认 `/bin/bash`）
- `login: bool` - 是否使用登录 Shell（默认 true）
- `yield_time_ms: Option<u64>` - 让出时间（毫秒）
- `max_output_tokens: Option<usize>` - 最大输出 token 数

### `WriteStdinArgs`

写入标准输入参数：
- `session_id: i32` - 会话 ID
- `chars: String` - 要写入的字符
- `yield_time_ms: Option<u64>` - 让出时间（毫秒）
- `max_output_tokens: Option<usize>` - 最大输出 token 数

## 主要函数

### `handle()`

处理统一执行请求：
1. 解析载荷（支持 `Function` 和 `UnifiedExec`）
2. 获取 `UnifiedExecSessionManager`
3. 创建 `UnifiedExecContext`
4. 根据工具名称分发：
   - `exec_command` → 调用 `manager.exec_command()`
   - `write_stdin` → 调用 `manager.write_stdin()`
5. 发送输出增量事件（如果有输出）
6. 格式化并返回响应

### `format_response()`

格式化 `UnifiedExecResponse` 为文本：
- Chunk ID（如果存在）
- 执行时长（秒）
- 退出码（如果进程已结束）
- 会话 ID（如果进程仍在运行）
- 原始 token 数量（如果存在）
- 输出内容

## 输出格式示例

### 命令完成

```
Wall time: 0.1234 seconds
Process exited with code 0
Output:
Hello, World!
```

### 命令仍在运行

```
Chunk ID: chunk-abc123
Wall time: 0.5000 seconds
Process running with session ID 42
Original token count: 150
Output:
Partial output...
```

## 工作流程

### 执行命令

1. AI 模型调用 `exec_command`
2. 发送 `UnifiedExecToolCallBegin` 事件
3. SessionManager 创建或重用 PTY 会话
4. 执行命令并收集输出
5. 如果有输出，发送 `ExecCommandOutputDelta` 事件
6. 返回结果（包含会话 ID 或退出码）

### 写入标准输入

1. AI 模型调用 `write_stdin` 并提供 `session_id`
2. SessionManager 向对应会话的 PTY 写入数据
3. 收集新的输出
4. 发送输出增量事件
5. 返回结果

## Yield Time 和 Token 限制

- **yield_time_ms**：在此时间后让出，返回当前输出
  - 允许长时间运行的命令分块返回输出
  - AI 模型可以根据中间输出做出决策

- **max_output_tokens**：限制返回的输出大小
  - 防止输出过长消耗过多 token
  - 超出部分会被截断，可能记录在 `original_token_count`

## 事件流

1. **UnifiedExecToolCallBegin**（由 emitter 发送）
   - 包含命令和工作目录

2. **ExecCommandOutputDelta**（由 handler 发送）
   - 包含 stdout 输出的增量数据
   - 实时流式传输给客户端

3. **ToolCallEnd**（由 emitter 发送）
   - 包含最终状态和结果

## 持久化会话

与普通 Shell 工具不同，Unified Exec 维护持久化的 PTY 会话：

**优势**：
- 保持环境变量和工作目录
- 支持交互式程序（如 REPL、调试器）
- 可以向运行中的进程发送输入
- 模拟真实的终端会话

**使用示例**：
```javascript
// 启动 Python REPL
exec_command({ cmd: "python3" })
// 返回 session_id: 123

// 向 REPL 输入代码
write_stdin({ session_id: 123, chars: "print('Hello')\n" })
// 返回输出: Hello

write_stdin({ session_id: 123, chars: "exit()\n" })
// 进程退出
```

## 与其他模块的关系

- **依赖模块**：
  - `unified_exec::UnifiedExecSessionManager` - 会话管理器
  - `unified_exec::UnifiedExecContext` - 执行上下文
  - `unified_exec::UnifiedExecResponse` - 响应类型

- **事件发送**：
  - `ToolEmitter::unified_exec()` - 统一执行事件发射器
  - `ExecCommandOutputDeltaEvent` - 输出增量事件

- **对比 ShellHandler**：
  - ShellHandler：一次性命令执行
  - UnifiedExecHandler：持久化交互式会话

## 错误处理

- 不支持的载荷：返回 "unified_exec handler received unsupported payload"
- 参数解析失败：返回 "failed to parse {tool} arguments"
- 执行失败：返回 "exec_command failed" 或 "write_stdin failed"
- 不支持的工具名：返回 "unsupported unified exec function {tool}"

## 设计特点

- **持久化**：维护长期运行的 Shell 会话
- **交互式**：支持向进程发送输入
- **流式输出**：增量返回输出，避免阻塞
- **Token 控制**：限制输出大小，优化 token 使用
- **时间控制**：支持 yield_time，实现渐进式处理
