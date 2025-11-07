# apply_patch.rs

## 文件作用

`apply_patch.rs` 实现了文件补丁应用功能，处理模型请求的文件修改操作。它负责评估补丁安全性、请求用户审批、并将补丁操作委托给执行层。

## 主要结构体

### InternalApplyPatchInvocation
补丁调用的内部表示。

**变体:**
- `Output(Result<String, FunctionCallError>)` - 直接返回结果（用于被拒绝的补丁）
- `DelegateToExec(ApplyPatchExec)` - 委托给exec执行（审批通过的补丁）

### ApplyPatchExec
委托执行的补丁信息。

**字段:**
- `action: ApplyPatchAction` - 补丁操作
- `user_explicitly_approved_this_action: bool` - 用户是否显式审批

## 主要函数

### apply_patch()
处理补丁应用请求的主函数。

**参数:**
- `sess: &Session` - 会话对象
- `turn_context: &TurnContext` - 轮次上下文
- `call_id: &str` - 调用ID
- `action: ApplyPatchAction` - 补丁操作

**返回:** `InternalApplyPatchInvocation` - 调用结果

**功能:**
1. 使用 `assess_patch_safety()` 评估补丁安全性
2. 根据安全检查结果：
   - `AutoApprove` - 委托给exec执行
   - `AskUser` - 请求用户审批
   - `Reject` - 拒绝并返回错误

### convert_apply_patch_to_protocol()
将内部补丁表示转换为协议格式。

**参数:**
- `action: &ApplyPatchAction` - 补丁操作

**返回:** `HashMap<PathBuf, FileChange>` - 路径到文件变更的映射

**功能:**
转换补丁变更类型（Add/Delete/Update）为协议定义的格式。

## 常量

- `CODEX_APPLY_PATCH_ARG1` - 应用补丁的命令行参数标识

## 与其他模块的关系

**依赖模块:**
- `codex::{Session, TurnContext}` - 会话管理
- `safety::{SafetyCheck, assess_patch_safety}` - 安全评估
- `codex_apply_patch::ApplyPatchAction` - 补丁操作定义

**被依赖模块:**
- `response_processing` - 处理模型响应时调用

**核心流程:**
模型请求修改文件 → 评估安全性 → 请求审批（如需） → 委托执行 → 返回结果
