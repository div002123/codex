# function_tool.rs

## 文件作用

`function_tool.rs` 定义工具调用错误类型。

## 主要枚举

### FunctionCallError
工具调用错误。

**变体:**
- `RespondToModel(String)` - 回复给模型的错误消息
- `Denied(String)` - 被拒绝的操作
- `MissingLocalShellCallId` - 缺少本地 Shell 调用 ID
- `Fatal(String)` - 致命错误

**实现:**
实现 `Error` trait，用于错误传播。

## 与其他模块的关系

**被依赖模块:**
- `apply_patch` - 返回补丁错误
- `response_processing` - 处理工具调用错误
- `tools` - 工具执行错误处理
