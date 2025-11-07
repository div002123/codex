# handlers/shell.rs

## 文件作用

实现 Shell 命令执行工具处理器，负责接收和处理 Shell 命令调用，并智能检测和处理 apply_patch 命令。

## 主要结构体

### `ShellHandler`

Shell 命令工具的处理器，实现 `ToolHandler` trait。

**特性**：
- 支持 `ToolPayload::Function` 和 `ToolPayload::LocalShell` 两种载荷类型
- 自动检测和拦截 apply_patch 命令
- 集成权限提升和批准流程

## 主要函数

### `to_exec_params()`

将 `ShellToolCallParams` 转换为 `ExecParams`：
- 解析命令和工作目录
- 创建环境变量
- 处理权限提升请求
- 设置超时参数

### `handle()`

处理工具调用的主入口：
1. 根据载荷类型解析参数：
   - `ToolPayload::Function` → 解析 JSON 参数
   - `ToolPayload::LocalShell` → 直接使用参数
2. 转换为 `ExecParams`
3. 调用 `run_exec_like()` 执行命令

### `run_exec_like()`

执行类 Shell 命令的核心函数：

**处理流程**：

1. **权限检查**：
   - 验证权限提升请求与批准策略的兼容性
   - 在非 OnRequest 模式下拒绝权限提升

2. **Apply Patch 检测**：
   使用 `codex_apply_patch::maybe_parse_apply_patch_verified()` 检测命令：

   - **Body**（有效补丁）：
     - 调用 `apply_patch::apply_patch()` 处理
     - 如果返回 `Output`：直接返回结果
     - 如果返回 `DelegateToExec`：委托给 `ApplyPatchRuntime` 执行

   - **CorrectnessError**：返回验证错误

   - **ShellParseError** / **NotApplyPatch**：继续执行 Shell 命令

3. **Shell 命令执行**：
   - 发送工具调用开始事件
   - 创建 `ShellRequest`
   - 使用 `ToolOrchestrator` 和 `ShellRuntime` 执行
   - 发送工具调用结束事件
   - 返回执行结果

## 参数

### `ShellToolCallParams`（来自协议）

- `command: Vec<String>` - 命令行参数数组
- `workdir: Option<String>` - 工作目录（相对或绝对路径）
- `timeout_ms: Option<u64>` - 超时时间（毫秒）
- `with_escalated_permissions: Option<bool>` - 是否请求权限提升
- `justification: Option<String>` - 权限提升的理由

## 执行路径

### 路径 1：Apply Patch（已批准）
```
Shell 命令 → 检测到 apply_patch → apply_patch::apply_patch() → 直接返回
```

### 路径 2：Apply Patch（需批准）
```
Shell 命令 → 检测到 apply_patch → DelegateToExec
    → ApplyPatchRuntime → ToolOrchestrator → 执行补丁
```

### 路径 3：普通 Shell 命令
```
Shell 命令 → ShellRequest → ToolOrchestrator → ShellRuntime → 执行命令
```

## 权限提升处理

当 `with_escalated_permissions: true` 时：
- **OnRequest 策略**：允许，将通过批准流程
- **其他策略**：拒绝，返回错误信息

错误消息示例：
```
approval policy is Never; reject command — you should not ask for
escalated permissions if the approval policy is Never
```

## 与其他模块的关系

- **依赖模块**：
  - `apply_patch` - 补丁应用逻辑
  - `codex_apply_patch` - 补丁解析和验证
  - `tools::runtimes::shell` - Shell 运行时
  - `tools::runtimes::apply_patch` - Apply Patch 运行时
  - `tools::orchestrator` - 工具编排器

- **事件发送**：
  - `ToolEmitter::shell()` - Shell 命令事件
  - `ToolEmitter::apply_patch()` - Apply Patch 事件

- **执行环境**：
  - `ExecParams` - 执行参数
  - `create_env()` - 环境变量创建
  - `ShellEnvironmentPolicy` - Shell 环境策略

## 设计特点

- **智能检测**：自动识别 apply_patch 命令
- **统一接口**：Shell 和 Apply Patch 使用相同的入口
- **权限安全**：严格的权限提升策略检查
- **灵活路由**：根据命令类型选择合适的执行路径
- **事件跟踪**：完整的执行事件记录

## 使用场景

- AI 模型执行系统命令（如 `git status`）
- AI 模型通过 Shell 语法应用代码补丁
- 用户通过 UI 手动执行 Shell 命令
- 需要权限提升的特殊操作
