# context_manager/normalize.rs

## 文件作用

提供历史记录规范化功能，确保工具调用和输出的完整性和一致性。

## 主要函数

### ensure_call_outputs_present

确保所有工具调用都有对应的输出。

**处理的调用类型：**

- `FunctionCall` - 函数调用
- `CustomToolCall` - 自定义工具调用
- `LocalShellCall` - 本地 shell 调用（如有 call_id）

**行为：**

- 检查每个调用是否有对应的输出
- 缺失输出时：
  - 记录错误或 panic（通过 `error_or_panic`）
  - 插入合成的 "aborted" 输出
  - 输出插入到调用条目之后

**合成输出：**

- `FunctionCallOutput { content: "aborted", ... }`
- `CustomToolCallOutput { output: "aborted" }`

### remove_orphan_outputs

移除没有对应调用的孤立输出。

**处理的输出类型：**

- `FunctionCallOutput` - 需要对应的 `FunctionCall` 或 `LocalShellCall`
- `CustomToolCallOutput` - 需要对应的 `CustomToolCall`

**行为：**

- 收集所有调用的 call_id
- 保留有匹配调用的输出
- 移除孤立输出，并记录错误

### remove_corresponding_for

移除与指定条目对应的配对条目。

**处理的配对关系：**

- `FunctionCall` ↔ `FunctionCallOutput`
- `CustomToolCall` ↔ `CustomToolCallOutput`
- `LocalShellCall` ↔ `FunctionCallOutput`

**用途：**

- 在 `remove_first_item` 后维护历史一致性
- 避免留下孤立的调用或输出

### remove_first_matching（私有）

移除第一个匹配谓词的条目。

**参数：**

- `items: &mut Vec<ResponseItem>` - 条目列表
- `predicate: F` - 匹配函数

## 与其他模块的关系

- 被 `ContextManager` 调用，维护历史一致性
- 依赖 `ResponseItem` 的各种变体
- 使用 `util::error_or_panic` 处理不一致情况
- 确保发送给模型的历史符合 API 要求
- 防止因缺失调用或输出导致的协议错误
- 支持历史记录的增量修改（移除条目时保持配对）
