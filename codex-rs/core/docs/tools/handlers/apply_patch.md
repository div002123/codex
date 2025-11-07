# handlers/apply_patch.rs

## 文件作用

实现 ApplyPatch 工具处理器，负责处理代码补丁的应用请求，支持文件的创建、删除、更新和移动操作。

## 主要结构体

### `ApplyPatchHandler`

应用补丁工具的处理器，实现 `ToolHandler` trait。

**特性**：
- 支持 Function 和 Custom 两种载荷类型
- 集成补丁验证机制
- 支持用户批准流程
- 可委托给运行时执行

### `ApplyPatchToolType`

补丁工具的类型枚举：
- `Freeform` - 自由格式（用于 GPT-5）
- `Function` - JSON 函数格式（用于 GPT-OSS）

## 主要函数

### `handle()`

异步处理补丁应用请求：
1. 解析工具载荷（支持 Function 和 Custom 格式）
2. 验证补丁格式和正确性
3. 根据验证结果：
   - 直接应用（已批准）
   - 委托给 ApplyPatchRuntime 执行（需批准）
   - 返回错误（验证失败）

### `create_apply_patch_freeform_tool()`

创建自由格式的补丁工具规范，使用 Lark 语法定义。

### `create_apply_patch_json_tool()`

创建 JSON 格式的补丁工具规范，包含完整的补丁语法说明：
- 支持添加文件（`*** Add File:`）
- 支持删除文件（`*** Delete File:`）
- 支持更新文件（`*** Update File:`）
- 支持文件移动（`*** Move to:`）
- 使用 hunk 格式表示代码变更

## 补丁格式

补丁使用专门设计的格式：

```
*** Begin Patch
*** Add File: <path>
+新文件内容

*** Update File: <path>
*** Move to: <new_path>  // 可选
@@ 可选的 hunk 头部
 上下文行
-删除的行
+添加的行

*** Delete File: <path>
*** End Patch
```

## 与其他模块的关系

- **依赖模块**：
  - `apply_patch` - 核心补丁应用逻辑
  - `tools::runtimes::apply_patch` - 补丁应用运行时
  - `tools::orchestrator` - 工具编排器
  - `codex_apply_patch` - 补丁解析和验证

- **协作模块**：
  - `ToolEmitter` - 发送工具事件
  - `ToolOrchestrator` - 管理工具执行流程
  - `ApplyPatchRuntime` - 实际执行补丁应用

- **使用场景**：AI 模型通过此处理器编辑文件内容
