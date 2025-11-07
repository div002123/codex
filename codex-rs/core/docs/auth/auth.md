# auth.rs - 认证模块

## 文件作用

该文件是 Codex 的核心认证模块，负责管理用户身份验证、令牌刷新、以及多种认证方式的支持（API Key 和 ChatGPT OAuth）。

## 主要结构体

### CodexAuth
- **作用**: 认证状态管理的核心结构
- **字段**:
  - `mode: AuthMode` - 认证模式（ApiKey 或 ChatGPT）
  - `api_key: Option<String>` - API 密钥
  - `auth_dot_json: Arc<Mutex<Option<AuthDotJson>>>` - 认证配置数据
  - `storage: Arc<dyn AuthStorageBackend>` - 存储后端
  - `client: CodexHttpClient` - HTTP 客户端

### AuthManager
- **作用**: 认证管理器，提供单一的认证数据源
- **功能**:
  - 缓存认证状态
  - 提供认证数据的重载机制
  - 支持令牌刷新和登出

### RefreshTokenError
- **作用**: 令牌刷新错误类型
- **变体**:
  - `Permanent(RefreshTokenFailedError)` - 永久性错误
  - `Transient(std::io::Error)` - 临时性错误

## 主要函数

### 认证加载
- `from_auth_storage()` - 从存储加载认证信息
- `load_auth()` - 内部加载函数，支持环境变量
- `read_openai_api_key_from_env()` - 从环境变量读取 OpenAI API Key
- `read_codex_api_key_from_env()` - 从环境变量读取 Codex API Key

### 令牌管理
- `refresh_token()` - 刷新访问令牌
- `get_token()` - 获取当前有效令牌
- `get_token_data()` - 获取完整的令牌数据
- `try_refresh_token()` - 尝试刷新令牌（内部函数）
- `update_tokens()` - 更新存储的令牌

### 用户信息
- `get_account_id()` - 获取账户 ID
- `get_account_email()` - 获取账户邮箱
- `account_plan_type()` - 获取账户计划类型（面向账户）
- `get_plan_type()` - 获取内部计划类型

### 登录/登出
- `login_with_api_key()` - 使用 API Key 登录
- `logout()` - 删除认证文件并登出
- `save_auth()` - 保存认证信息
- `load_auth_dot_json()` - 加载认证配置

### 安全与限制
- `enforce_login_restrictions()` - 强制执行登录限制
- `classify_refresh_token_failure()` - 分类令牌刷新失败原因

## 常量

- `TOKEN_REFRESH_INTERVAL: i64 = 8` - 令牌刷新间隔（天）
- `REFRESH_TOKEN_URL` - 令牌刷新端点 URL
- `CLIENT_ID` - OAuth 客户端 ID
- `OPENAI_API_KEY_ENV_VAR` - OpenAI API Key 环境变量名
- `CODEX_API_KEY_ENV_VAR` - Codex API Key 环境变量名

## 与其他模块的关系

- **依赖 `auth/storage.rs`**: 使用存储后端接口保存和加载认证数据
- **依赖 `token_data.rs`**: 解析和管理 JWT 令牌信息
- **依赖 `config.rs`**: 使用配置信息进行认证限制
- **被 CLI 使用**: 为命令行工具提供认证服务
- **被 API 客户端使用**: 为 HTTP 请求提供认证令牌
