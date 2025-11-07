# safety.rs - 安全评估模块

## 文件作用

该文件实现了补丁（patch）安全性评估和平台沙箱检测功能，决定文件修改操作是否需要用户批准以及是否可以自动在沙箱中执行。

## 主要枚举

### SafetyCheck
- **作用**: 安全检查的结果
- **变体**:
  - `AutoApprove { sandbox_type, user_explicitly_approved }` - 自动批准
  - `AskUser` - 需要询问用户
  - `Reject { reason }` - 拒绝执行

## 主要函数

### assess_patch_safety
- **签名**: `pub fn assess_patch_safety(action: &ApplyPatchAction, policy: AskForApproval, sandbox_policy: &SandboxPolicy, cwd: &Path) -> SafetyCheck`
- **作用**: 评估补丁操作的安全性
- **检查流程**:
  1. 空补丁检查 - 拒绝空操作
  2. 策略检查 - 根据 `AskForApproval` 决定基本行为
  3. 路径约束检查 - 验证文件路径是否在可写范围内
  4. 沙箱可用性检查 - 确定是否可以安全执行

### get_platform_sandbox
- **签名**: `pub fn get_platform_sandbox() -> Option<SandboxType>`
- **作用**: 检测当前平台可用的沙箱类型
- **返回值**:
  - macOS → `Some(SandboxType::MacosSeatbelt)`
  - Linux → `Some(SandboxType::LinuxSeccomp)`
  - Windows → `Some(SandboxType::WindowsRestrictedToken)` (如果启用)
  - 其他 → `None`

### is_write_patch_constrained_to_writable_paths
- **签名**: `fn is_write_patch_constrained_to_writable_paths(...) -> bool`
- **作用**: 检查补丁的所有文件操作是否限制在可写路径内
- **检查内容**:
  - Add 操作 - 目标路径
  - Delete 操作 - 目标路径
  - Update 操作 - 源路径和目标路径（如果有移动）

## AskForApproval 策略处理

### OnFailure
- **行为**: 总是自动批准（在沙箱中执行）
- **失败时**: 沙箱拒绝则回退询问用户

### Never
- **行为**:
  - 路径受限且有沙箱 → 自动批准
  - 路径不受限 → 拒绝执行

### OnRequest
- **行为**:
  - 路径受限 → 自动批准
  - 路径不受限 → 询问用户

### UnlessTrusted
- **行为**: 总是询问用户（除非路径受限）

## 沙箱策略处理

### DangerFullAccess
- **行为**: 完全跳过沙箱，直接执行
- **风险**: 不受限制，需谨慎使用

### ReadOnly
- **行为**: 写操作总是不受约束（策略不允许写入）
- **结果**: 需要用户批准或拒绝

### WorkspaceWrite
- **检查**: 验证所有路径是否在 `writable_roots` 内
- **成功**: 可以在沙箱中自动执行

## 路径规范化

### normalize 函数
- **作用**: 不访问文件系统的路径规范化
- **处理**:
  - 移除 `.` (当前目录)
  - 解析 `..` (父目录)
  - 保留其他路径组件
- **特点**: 即使文件不存在也能工作

### 路径匹配逻辑
1. 转换为绝对路径（基于 cwd）
2. 规范化路径
3. 检查是否在任一可写根目录内
4. 考虑只读子路径排除

## Windows 沙箱功能

### 功能开关
- **全局标志**: `WINDOWS_SANDBOX_ENABLED: AtomicBool`
- **控制函数**: `set_windows_sandbox_enabled(enabled: bool)`
- **原因**: Windows 沙箱是实验性功能，需要显式启用

### 非 Windows 平台
- 提供 no-op 实现
- 函数存在但不执行任何操作

## 安全考虑

### 硬链接风险
- **问题**: 补丁路径可能是硬链接，指向可写根之外
- **缓解**: 即使路径检查通过，仍在沙箱中执行
- **理念**: 深度防御，多层保护

### 自动批准条件
1. **路径受限**: 所有操作在可写范围内
2. **沙箱可用**: 平台支持沙箱隔离
3. **策略允许**: `AskForApproval` 策略不强制询问

### 拒绝条件
- 空补丁
- `AskForApproval::Never` + 路径不受限
- 路径不受限 + 无可用沙箱

## 文件变更类型处理

### Add 操作
- 检查: 新文件路径是否可写

### Delete 操作
- 检查: 删除文件路径是否在可写范围

### Update 操作
- 检查:
  - 原文件路径是否可写
  - 如果有 move_path，目标路径是否可写

## 测试覆盖

### 测试场景
1. 内部文件操作（应通过）
2. 外部文件操作（应阻止）
3. 添加可写根后的外部操作（应通过）
4. 临时目录排除逻辑

## 与其他模块的关系

- **依赖 `codex_apply_patch`**: 使用 `ApplyPatchAction` 和 `ApplyPatchFileChange`
- **依赖 `protocol::SandboxPolicy`**: 使用沙箱策略定义
- **依赖 `protocol::AskForApproval`**: 使用审批策略
- **依赖 `exec::SandboxType`**: 返回沙箱类型
- **被 `tools` 模块使用**: 决定补丁执行策略
- **被 `sandboxing` 模块使用**: 提供沙箱类型检测
- **与平台代码协作**: 根据操作系统返回不同沙箱
