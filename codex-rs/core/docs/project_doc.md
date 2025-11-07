# project_doc.rs

## 文件作用

`project_doc.rs` 管理项目文档和用户指令，从 AGENTS.md 等文件加载项目特定的指令。

## 主要函数

### get_user_instructions()
获取用户指令。

**参数:**
- `config: &Config` - 配置

**返回:** `Option<String>` - 用户指令文本

**功能:**
- 从 AGENTS.md 加载指令
- 从配置文件加载指令
- 合并多个来源的指令

## 与其他模块的关系

**依赖模块:**
- `config::Config` - 配置管理

**被依赖模块:**
- `codex::Session` - 构建初始上下文时使用
