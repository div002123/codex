# user_instructions.rs

## 文件作用

`user_instructions.rs` 处理用户指令的加载和格式化，包括 AGENTS.md 和配置中的指令。

## 主要结构体

### UserInstructions
用户指令。

**方法:**
- `is_user_instructions()` - 检查消息是否为用户指令
- `parse()` - 解析用户指令

### DeveloperInstructions
开发者指令。

**功能:**
提供开发者级别的系统指令。

## 与其他模块的关系

**被依赖模块:**
- `codex::Session` - 构建初始上下文
- `event_mapping` - 过滤用户指令消息
