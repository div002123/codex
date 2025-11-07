# handlers/plan.rs

## 文件作用

实现计划（Plan）工具处理器，允许 AI 模型创建和更新任务计划，为客户端提供结构化的进度跟踪。

## 主要结构体

### `PlanHandler`

计划工具的处理器，实现 `ToolHandler` trait。

## 主要常量

### `PLAN_TOOL`

类型：`LazyLock<ToolSpec>`

计划工具的完整规范定义，包含：

**参数结构**：
- `explanation: String` - 计划说明（可选）
- `plan: Array<PlanItem>` - 计划步骤列表（必需）
  - `step: String` - 步骤描述
  - `status: String` - 状态："pending"、"in_progress" 或 "completed"

**约束**：
- 同一时间最多只能有一个步骤处于 `in_progress` 状态
- `plan` 字段为必需字段
- 不允许额外的字段（`strict: false` 但 `additional_properties: false`）

## 主要函数

### `handle()`

处理计划更新请求：
1. 验证载荷类型（仅支持 `ToolPayload::Function`）
2. 提取参数
3. 调用 `handle_update_plan()` 处理计划更新
4. 返回成功响应 "Plan updated"

### `handle_update_plan()`

实际处理计划更新的函数：
1. 解析参数为 `UpdatePlanArgs`
2. 通过 `EventMsg::PlanUpdate` 发送计划更新事件
3. 返回 "Plan updated" 字符串

**注意**：此函数本身不执行有用的操作，其价值在于：
- 强制 AI 模型结构化地记录计划
- 为客户端提供可读取和渲染的计划数据
- 输入比输出更重要（客户端读取输入，而非输出）

### `parse_update_plan_arguments()`

解析 JSON 参数字符串为 `UpdatePlanArgs` 结构。

## 工具描述

```
Updates the task plan.
Provide an optional explanation and a list of plan items, each with a step and status.
At most one step can be in_progress at a time.
```

## 使用示例

```json
{
  "explanation": "处理文件上传功能",
  "plan": [
    { "step": "创建上传端点", "status": "completed" },
    { "step": "实现文件验证", "status": "in_progress" },
    { "step": "添加进度跟踪", "status": "pending" },
    { "step": "编写测试", "status": "pending" }
  ]
}
```

## 与其他模块的关系

- **依赖模块**：
  - `codex_protocol::plan_tool::UpdatePlanArgs` - 计划参数定义
  - `EventMsg::PlanUpdate` - 计划更新事件

- **事件流**：
  ```
  AI 模型 → PlanHandler → EventMsg::PlanUpdate → 客户端
  ```

- **使用场景**：
  - AI 模型记录其工作计划
  - 客户端显示进度和当前步骤
  - 用户了解 AI 的执行策略

- **设计理念**：
  - 工具的价值在于输入而非输出
  - 为客户端提供结构化的 AI 意图
  - 帮助 AI 模型组织其思维过程

## 特点

- **声明式**：工具本身不改变系统状态
- **可观察性**：为客户端提供 AI 工作的可见性
- **结构化**：强制使用标准格式记录计划
- **实时更新**：支持计划的动态更新
