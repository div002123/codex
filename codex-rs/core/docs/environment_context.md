# environment_context.rs

## 文件作用

`environment_context.rs` 定义了环境上下文结构，用于向模型传递会话的运行环境信息（工作目录、沙箱策略、审批策略等），并支持增量更新。

## 主要结构体

### EnvironmentContext
环境上下文信息。

**字段:**
- `cwd: Option<PathBuf>` - 当前工作目录
- `approval_policy: Option<AskForApproval>` - 审批策略
- `sandbox_mode: Option<SandboxMode>` - 沙箱模式
- `network_access: Option<NetworkAccess>` - 网络访问权限
- `writable_roots: Option<Vec<PathBuf>>` - 可写根目录列表
- `shell: Option<Shell>` - Shell 类型

**方法:**

#### new()
创建新的环境上下文。

#### equals_except_shell()
比较两个环境上下文（忽略shell字段）。

**用途:** 判断轮次之间环境是否变化。

#### diff()
计算两个轮次上下文之间的差异。

**返回:** 仅包含变化字段的 `EnvironmentContext`

**用途:** 增量更新，避免重复发送相同信息。

#### serialize_to_xml()
序列化为XML格式。

**格式:**
```xml
<environment_context>
  <cwd>/path/to/dir</cwd>
  <approval_policy>...</approval_policy>
  <sandbox_mode>...</sandbox_mode>
  <network_access>...</network_access>
  <writable_roots>
    <root>/path/1</root>
    <root>/path/2</root>
  </writable_roots>
  <shell>bash</shell>
</environment_context>
```

### NetworkAccess
网络访问权限枚举。

**变体:**
- `Restricted` - 受限制
- `Enabled` - 已启用

## 类型转换

### From<&TurnContext>
从轮次上下文创建环境上下文（不包含shell）。

### From<EnvironmentContext> for ResponseItem
转换为用户消息，内容为XML格式的环境上下文。

## 与其他模块的关系

**依赖模块:**
- `codex::TurnContext` - 轮次上下文
- `protocol::{AskForApproval, SandboxPolicy}` - 策略定义
- `shell::Shell` - Shell 类型

**被依赖模块:**
- `codex::Session` - 构建初始上下文时使用
- 会话历史 - 作为消息发送给模型

**核心流程:**
轮次开始 → 生成环境上下文 → 与前一轮次对比 → 仅发送变化部分 → 模型获知当前环境
