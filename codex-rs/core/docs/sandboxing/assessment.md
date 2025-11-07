# sandboxing/assessment.rs - 沙箱命令风险评估

## 文件作用

该文件实现了使用 AI 模型对沙箱命令进行智能风险评估的功能，提供命令安全性的自动化分析。

## 主要结构体

### SandboxAssessmentPromptTemplate
- **作用**: 用于生成评估提示的模板
- **字段**:
  - `platform: &str` - 操作系统平台
  - `sandbox_policy: &str` - 沙箱策略摘要
  - `filesystem_roots: Option<&str>` - 文件系统根路径
  - `working_directory: &str` - 工作目录
  - `command_argv: &str` - 命令参数（JSON 格式）
  - `command_joined: &str` - 命令字符串（shell 格式）
  - `sandbox_failure_message: Option<&str>` - 沙箱失败消息

## 主要函数

### assess_command
- **签名**: `pub(crate) async fn assess_command(...) -> Option<SandboxCommandAssessment>`
- **作用**: 评估命令的风险级别
- **参数**:
  - `config` - 配置对象
  - `provider` - 模型提供商信息
  - `auth_manager` - 认证管理器
  - `parent_otel` - 遥测事件管理器
  - `conversation_id` - 会话 ID
  - `session_source` - 会话来源
  - `call_id` - 调用 ID
  - `command` - 待评估的命令
  - `sandbox_policy` - 沙箱策略
  - `cwd` - 当前工作目录
  - `failure_message` - 失败消息（可选）

### 评估流程

1. **配置检查**
   - 检查 `experimental_sandbox_command_assessment` 功能是否启用
   - 验证命令非空

2. **准备上下文**
   - 序列化命令为 JSON
   - 使用 shlex 格式化命令字符串
   - 收集文件系统根路径
   - 获取平台信息（OS）

3. **生成提示**
   - 使用 Askama 模板引擎
   - 分离系统提示和用户提示
   - 包含沙箱策略、路径和命令详情

4. **调用 AI 模型**
   - 使用 ModelClient 创建流式请求
   - 应用输出模式约束（JSON Schema）
   - 超时时间：5 秒

5. **解析结果**
   - 提取 JSON 响应
   - 反序列化为 `SandboxCommandAssessment`
   - 记录遥测数据

### summarize_sandbox_policy
- **作用**: 将沙箱策略转换为人类可读的摘要
- **输出格式**:
  - `"danger-full-access"` - 完全访问
  - `"read-only"` - 只读
  - `"workspace-write (network_access=...)"` - 工作区写入

### sandbox_roots_for_prompt
- **作用**: 提取沙箱策略中的文件系统根路径
- **包含**: 当前工作目录和可写根路径

### sandbox_assessment_schema
- **作用**: 定义 AI 模型输出的 JSON Schema
- **结构**:
  ```json
  {
    "description": "命令描述（1-500字符）",
    "risk_level": "low|medium|high"
  }
  ```

### response_item_text
- **作用**: 从响应项中提取文本内容
- **支持**: Message、FunctionCallOutput 等类型

## 常量

- `SANDBOX_ASSESSMENT_TIMEOUT: Duration = 5秒` - 评估超时时间

## 遥测集成

### 记录的事件
- `sandbox_assessment_latency()` - 评估延迟
- `sandbox_assessment()` - 评估结果，包括：
  - 状态：success / parse_error / no_output / model_error / timeout
  - 风险级别：low / medium / high（成功时）
  - 持续时间

## 错误处理

### 失败场景
1. **模板渲染失败** - 记录警告，返回 None
2. **提示格式错误** - 缺少分隔符，返回 None
3. **模型调用失败** - 记录警告，标记为 "model_error"
4. **超时** - 5 秒后取消，标记为 "timeout"
5. **JSON 解析失败** - 记录警告，标记为 "parse_error"
6. **无输出** - 模型未返回消息，标记为 "no_output"

### 容错设计
- 所有失败情况返回 `None` 而非错误
- 评估失败不阻塞命令执行
- 仅在功能启用时执行评估

## 使用场景

1. **自动风险评估**: 在执行前评估命令风险
2. **审计日志**: 记录命令的安全性评估
3. **用户提示**: 向用户显示风险级别
4. **策略执行**: 基于风险级别决定是否需要额外确认

## 与其他模块的关系

- **依赖 `client::ModelClient`**: 调用 AI 模型
- **依赖 `protocol::SandboxPolicy`**: 使用沙箱策略
- **依赖 `auth::AuthManager`**: 获取认证令牌
- **依赖 `config::Config`**: 读取配置和模型设置
- **集成 `codex_otel`**: 记录遥测数据
- **被 `exec` 或 `tools` 调用**: 在命令执行前评估
- **使用 Askama 模板**: 生成结构化提示
