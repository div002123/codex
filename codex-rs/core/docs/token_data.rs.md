# token_data.rs

## 文件作用

`token_data.rs` 处理JWT（JSON Web Token）的解析和认证数据的管理。该文件定义了存储和解析OpenAI认证token的数据结构，包括从JWT中提取用户信息（如email、订阅计划类型、账户ID等）。它实现了JWT payload的Base64解码和JSON解析，并提供了自定义的序列化/反序列化逻辑。

文件特别关注ChatGPT订阅计划信息的提取，支持识别已知的计划类型（Free、Plus、Pro、Team、Business、Enterprise、Edu）以及未知的计划类型。

## 主要结构体列表

- `TokenData` - Token数据容器
  - `id_token` - ID token信息（JWT解析后）
  - `access_token` - 访问token（JWT字符串）
  - `refresh_token` - 刷新token
  - `account_id` - 账户ID（可选）

- `IdTokenInfo` - ID token中的关键信息
  - `email` - 用户email（可选）
  - `chatgpt_plan_type` - ChatGPT订阅计划类型（可选）
  - `chatgpt_account_id` - ChatGPT账户ID（可选）
  - `raw_jwt` - 原始JWT字符串

- `PlanType` - 订阅计划类型（枚举）
  - `Known(KnownPlan)` - 已知的计划类型
  - `Unknown(String)` - 未知的计划类型（保留原始字符串）

- `KnownPlan` - 已知的订阅计划（枚举）
  - Free - 免费计划
  - Plus - Plus计划
  - Pro - Pro计划
  - Team - Team计划
  - Business - Business计划
  - Enterprise - Enterprise计划
  - Edu - 教育计划

### 私有结构体：
- `IdClaims` - JWT claims结构
- `AuthClaims` - OpenAI特定的认证claims

## 主要函数/方法列表

### IdTokenInfo 方法：
- `get_chatgpt_plan_type()` - 获取计划类型的字符串表示

### 模块函数：
- `parse_id_token()` - 解析JWT字符串并提取信息
  - 分割JWT（header.payload.signature）
  - Base64解码payload
  - 解析JSON claims
  - 提取用户信息

### 自定义序列化：
- `deserialize_id_token()` - 自定义反序列化器，从JWT字符串解析为IdTokenInfo
- `serialize_id_token()` - 自定义序列化器，将IdTokenInfo序列化为JWT字符串

## 与其他模块的关系

- **被 auth 使用**: 认证模块使用 `TokenData` 存储和管理用户凭证
- **被 error 使用**: `error.rs` 使用 `PlanType` 和 `KnownPlan` 来生成计划特定的错误消息
- **被 client 使用**: 间接通过 `auth` 模块，为API请求提供认证信息
- **JWT处理**: 使用 base64 crate 进行Base64解码
- **序列化支持**: 使用 serde 进行JSON和结构体的转换
- **错误处理**: 定义了 `IdTokenInfoError` 用于JWT解析错误
- **安全存储**: 这些结构通常被序列化到 auth.json 文件中
- **订阅管理**: 计划类型信息用于确定用户的访问权限和使用限制
- **OpenAI协议**: 遵循OpenAI的JWT claims格式（特别是 "https://api.openai.com/auth" 命名空间）
- **类型安全**: 通过枚举类型提供类型安全的计划识别，同时保持向后兼容性（Unknown变体）
