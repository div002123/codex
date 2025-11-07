# response_processing.rs

## 文件作用

`response_processing.rs` 处理模型响应，包括工具调用、流式输出、错误处理等。

## 主要函数

### process_items()
处理响应项列表。

**参数:**
- `sess: &Session` - 会话
- `turn_context: &TurnContext` - 轮次上下文
- `items: Vec<ResponseItem>` - 响应项

**功能:**
- 处理工具调用（MCP工具、本地工具）
- 执行命令审批
- 应用补丁
- 聚合结果
- 发送事件

## 与其他模块的关系

**依赖模块:**
- `codex::{Session, TurnContext}` - 会话管理
- `mcp_tool_call::handle_mcp_tool_call` - MCP工具调用
- `apply_patch::apply_patch` - 补丁应用

**被依赖模块:**
- `codex::submission_loop` - 主循环中处理响应
