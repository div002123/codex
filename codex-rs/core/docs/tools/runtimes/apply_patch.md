# runtimes/apply_patch.rs

## 文件作用

实现 Apply Patch 运行时，在编排器（Orchestrator）下执行已验证的代码补丁。假定补丁的验证和批准已在上游完成，复用批准决策以避免重复提示。

## 主要结构体

### `ApplyPatchRequest`

补丁应用请求：
- `patch: String` - 补丁内容
- `cwd: PathBuf` - 工作目录
- `timeout_ms: Option<u64>` - 超时时间（毫秒）
- `user_explicitly_approved: bool` - 用户是否已明确批准
- `codex_exe: Option<PathBuf>` - codex 可执行文件路径

**Trait 实现**：
- `ProvidesSandboxRetryData`：返回 `None`（补丁不支持重试数据）

### `ApplyPatchRuntime`

补丁应用运行时。

**Trait 实现**：
- `ToolRuntime<ApplyPatchRequest, ExecToolCallOutput>`
- `Sandboxable`
- `Approvable<ApplyPatchRequest>`

### `ApprovalKey`

批准缓存键：
- `patch: String` - 补丁内容
- `cwd: PathBuf` - 工作目录

用于批准缓存，避免重复批准相同的补丁。

## 主要函数

### `build_command_spec()`

构建补丁执行的命令规范：
1. 确定 codex 可执行文件路径：
   - 优先使用 `req.codex_exe`
   - 否则使用 `std::env::current_exe()`
2. 构建命令：`codex --codex-run-as-apply-patch <patch>`
3. 设置工作目录和超时
4. 使用最小环境变量（空 HashMap）

**目的**：
- 自调用执行补丁
- 使用最小环境确保确定性
- 防止环境变量泄露

### `stdout_stream()`

创建标准输出流，用于实时输出补丁执行过程：
- `sub_id` - 订阅 ID
- `call_id` - 调用 ID
- `tx_event` - 事件发送器

## Trait 实现

### `Sandboxable`

沙箱行为配置：
- `sandbox_preference()` → `Auto`：自动选择沙箱策略
- `escalate_on_failure()` → `true`：失败时升级到无沙箱模式

### `Approvable<ApplyPatchRequest>`

批准逻辑配置：

#### `approval_key()`
返回用于批准缓存的键（patch + cwd）。

#### `start_approval_async()`

异步启动批准流程：

**逻辑**：
1. 如果有重试原因（retry_reason）：
   - 调用 `request_command_approval()` 请求批准
   - 显示重试原因和风险评估

2. 如果用户已明确批准（user_explicitly_approved）：
   - 返回 `ReviewDecision::ApprovedForSession`
   - 整个会话期间有效

3. 否则：
   - 返回 `ReviewDecision::Approved`
   - 仅当前操作有效

**批准缓存**：
使用 `with_cached_approval()` 避免重复批准相同的补丁。

#### `wants_no_sandbox_approval()`

确定无沙箱模式是否需要批准：
- `AskForApproval::Never` → 不需要批准
- 其他策略 → 需要批准

### `ToolRuntime<ApplyPatchRequest, ExecToolCallOutput>`

#### `run()`

执行补丁应用：
1. 调用 `build_command_spec()` 构建命令
2. 从 `SandboxAttempt` 获取执行环境
3. 使用 `execute_env()` 执行命令
4. 返回执行结果（`ExecToolCallOutput`）

**参数**：
- `req: &ApplyPatchRequest` - 补丁请求
- `attempt: &SandboxAttempt<'_>` - 沙箱尝试上下文
- `ctx: &ToolCtx<'_>` - 工具上下文

## 执行流程

```
ApplyPatchHandler
    ↓
创建 ApplyPatchRequest
    ↓
ToolOrchestrator::run()
    ↓
检查批准缓存
    ↓ 未缓存
start_approval_async()
    ↓ 批准通过
创建 SandboxAttempt
    ↓
ApplyPatchRuntime::run()
    ↓
build_command_spec()
    ↓
execute_env()
    ↓ 执行: codex --codex-run-as-apply-patch
    ↓
返回 ExecToolCallOutput
    ↓
格式化响应
    ↓
返回给 AI 模型
```

## 批准决策矩阵

| 场景 | user_explicitly_approved | retry_reason | 决策 |
|------|-------------------------|--------------|------|
| 首次执行（用户已批准） | true | None | ApprovedForSession |
| 首次执行（自动批准） | false | None | Approved |
| 重试执行 | * | Some(reason) | 请求新批准 |

## 安全特性

### 最小环境

使用空环境变量映射：
```rust
env: HashMap::new()
```

**好处**：
- 确保补丁执行的确定性
- 防止敏感环境变量泄露
- 减少外部依赖

### 自调用执行

使用 codex 自身作为补丁执行器：
```
codex --codex-run-as-apply-patch <patch_content>
```

**好处**：
- 复用 codex 的补丁解析和应用逻辑
- 确保一致的补丁语义
- 简化沙箱配置

## 与其他模块的关系

- **调用者**：
  - `handlers::apply_patch::ApplyPatchHandler`

- **依赖模块**：
  - `sandboxing::execute_env()` - 执行环境
  - `exec::ExecToolCallOutput` - 执行输出
  - `tools::orchestrator::ToolOrchestrator` - 编排器

- **批准系统**：
  - `Session::request_command_approval()` - 请求批准
  - `with_cached_approval()` - 批准缓存

## 错误处理

- 无法确定 codex 路径：返回 `ToolError::Rejected`
- 沙箱环境创建失败：返回 `ToolError::Codex`
- 补丁执行失败：返回 `ToolError::Codex`

## 设计特点

- **批准优化**：缓存批准决策，避免重复提示
- **会话级批准**：用户明确批准后，整个会话有效
- **确定性执行**：最小环境确保可重复性
- **实时输出**：通过 stdout_stream 流式传输输出
- **沙箱支持**：自动选择合适的沙箱策略
