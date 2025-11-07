# auth/storage.rs - 认证存储模块

## 文件作用

该文件实现了认证信息的持久化存储，支持多种存储后端（文件系统、系统密钥链），提供了灵活的凭证存储机制。

## 主要结构体

### AuthCredentialsStoreMode
- **作用**: 定义认证凭证的存储模式
- **变体**:
  - `File` - 存储在文件系统（默认）
  - `Keyring` - 存储在系统密钥链
  - `Auto` - 优先使用密钥链，失败时回退到文件

### AuthDotJson
- **作用**: `auth.json` 文件的数据结构
- **字段**:
  - `openai_api_key: Option<String>` - OpenAI API 密钥
  - `tokens: Option<TokenData>` - OAuth 令牌数据
  - `last_refresh: Option<DateTime<Utc>>` - 上次刷新时间

### FileAuthStorage
- **作用**: 文件系统存储后端
- **功能**: 在 `CODEX_HOME/auth.json` 中读写认证信息

### KeyringAuthStorage
- **作用**: 系统密钥链存储后端
- **功能**: 使用操作系统密钥链服务存储凭证
- **特点**: 更安全，避免明文存储

### AutoAuthStorage
- **作用**: 自动选择存储后端
- **策略**:
  - 优先尝试使用密钥链
  - 密钥链不可用时回退到文件存储
  - 提供无缝的存储体验

## 主要 Trait

### AuthStorageBackend
- **作用**: 存储后端的统一接口
- **方法**:
  - `load()` - 加载认证信息
  - `save()` - 保存认证信息
  - `delete()` - 删除认证信息

## 主要函数

### 存储创建
- `create_auth_storage()` - 根据模式创建存储后端
- `create_auth_storage_with_keyring_store()` - 使用指定密钥链创建存储

### 文件操作
- `get_auth_file()` - 获取认证文件路径
- `delete_file_if_exists()` - 删除认证文件（如果存在）

### 密钥链操作
- `compute_store_key()` - 计算密钥链存储键（基于路径哈希）
- `load_from_keyring()` - 从密钥链加载
- `save_to_keyring()` - 保存到密钥链

### 文件存储实现 (FileAuthStorage)
- `try_read_auth_json()` - 读取并解析 auth.json 文件
- `load()` - 实现 trait 的加载方法
- `save()` - 实现 trait 的保存方法（使用 0o600 权限）
- `delete()` - 实现 trait 的删除方法

## 常量

- `KEYRING_SERVICE: &str = "Codex Auth"` - 密钥链服务名称

## 安全特性

1. **文件权限**: Unix 系统上使用 0o600 权限（仅所有者可读写）
2. **密钥链集成**: 支持操作系统级别的安全存储
3. **路径哈希**: 密钥链键使用路径 SHA256 哈希，避免冲突
4. **自动清理**: 密钥链保存成功后删除旧的文件存储

## 与其他模块的关系

- **被 `auth.rs` 使用**: 提供认证数据的持久化能力
- **依赖 `codex_keyring_store`**: 使用密钥链抽象层
- **依赖 `token_data.rs`**: 序列化和反序列化令牌数据
- **被配置系统使用**: 根据配置选择存储模式
