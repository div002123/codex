# flags.rs

## 文件作用

`flags.rs` 定义环境变量标志，用于测试和调试。

## 环境变量

### CODEX_RS_SSE_FIXTURE
离线测试的 SSE 数据文件路径。

**类型:** `Option<&str>`
**默认值:** `None`

**用途:**
在测试中使用预录制的 SSE 响应，避免真实的 API 调用。

## 实现

使用 `env_flags` 宏自动从环境变量生成标志常量。

## 与其他模块的关系

**被依赖模块:**
- `client` - 测试时使用 fixture 数据
