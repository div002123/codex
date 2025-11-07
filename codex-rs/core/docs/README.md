# Codex Core 文档中心

欢迎来到 Codex Core 的完整中文文档！本文档涵盖了 `codex-rs/core` 模块的所有核心功能。

## 📋 快速导航

- [架构概览](./ARCHITECTURE.md) - 整体架构设计和流程图
- [模块索引](#模块索引) - 按功能分类的完整模块列表

---

## 🏗️ 架构概览

查看 [ARCHITECTURE.md](./ARCHITECTURE.md) 了解：
- 整体架构流程图（Mermaid）
- 核心数据流
- 模块职责划分
- 关键设计模式
- 安全架构
- 扩展点说明

---

## 📚 模块索引

### 1️⃣ 核心入口模块

| 文档 | 说明 |
|------|------|
| [lib.rs](./lib.rs.md) | 库根模块，组织和导出所有公共 API |

### 2️⃣ 会话与对话管理

| 文档 | 说明 |
|------|------|
| [conversation_manager.md](./conversation_manager.md) | 会话管理器，管理对话生命周期 |
| [codex_conversation.md](./codex_conversation.md) | 会话接口，提供对话交互 API |
| [codex.md](./codex.md) | Codex 主模块，核心业务逻辑 |
| [codex/compact.md](./codex/compact.md) | 会话压缩功能 |
| [codex_delegate.md](./codex_delegate.md) | 子代理（SubAgent）实现 |

### 3️⃣ 状态管理

| 文档 | 说明 |
|------|------|
| [state/mod.md](./state/mod.md) | 状态模块入口 |
| [state/service.md](./state/service.md) | SessionServices，管理会话服务依赖 |
| [state/session.md](./state/session.md) | SessionState，会话级别状态 |
| [state/turn.md](./state/turn.md) | ActiveTurn 和 TurnState，轮次状态管理 |

### 4️⃣ 任务系统

| 文档 | 说明 |
|------|------|
| [tasks/mod.md](./tasks/mod.md) | 任务系统核心，定义 SessionTask trait |
| [tasks/regular.md](./tasks/regular.md) | RegularTask，处理常规对话 |
| [tasks/compact.md](./tasks/compact.md) | CompactTask，对话历史压缩 |
| [tasks/review.md](./tasks/review.md) | ReviewTask，代码审查任务 |
| [tasks/undo.md](./tasks/undo.md) | UndoTask，撤销操作 |
| [tasks/ghost_snapshot.md](./tasks/ghost_snapshot.md) | GhostSnapshotTask，Git 快照 |
| [tasks/user_shell.md](./tasks/user_shell.md) | UserShellCommandTask，执行用户 Shell 命令 |

### 5️⃣ AI 客户端与通信

| 文档 | 说明 |
|------|------|
| [client.rs.md](./client.rs.md) | ModelClient，与 AI API 通信的核心客户端 |
| [client_common.rs.md](./client_common.rs.md) | 客户端通用类型，Prompt 和工具规范 |
| [chat_completions.md](./chat_completions.md) | Chat Completions API 实现 |
| [response_processing.md](./response_processing.md) | 响应处理和解析 |
| [default_client.md](./default_client.md) | HTTP 客户端配置 |

### 6️⃣ 模型配置

| 文档 | 说明 |
|------|------|
| [model_family.rs.md](./model_family.rs.md) | ModelFamily，模型家族配置 |
| [model_provider_info.rs.md](./model_provider_info.rs.md) | ModelProviderInfo，模型提供者注册表 |
| [openai_model_info.rs.md](./openai_model_info.rs.md) | OpenAI 模型元数据和规格 |

### 7️⃣ 认证与授权

| 文档 | 说明 |
|------|------|
| [auth/auth.md](./auth/auth.md) | 核心认证模块，AuthManager 和 CodexAuth |
| [auth/storage.md](./auth/storage.md) | 认证存储，支持文件和密钥链 |
| [token_data.rs.md](./token_data.rs.md) | JWT 解析和认证数据管理 |

### 8️⃣ 配置系统

| 文档 | 说明 |
|------|------|
| [config/mod.md](./config/mod.md) | 配置系统核心，Config 结构体 |
| [config/types.md](./config/types.md) | 配置数据类型定义 |
| [config/edit.md](./config/edit.md) | 配置编辑和持久化 |
| [config/profile.md](./config/profile.md) | ConfigProfile，配置文件管理 |
| [config_loader/mod.md](./config_loader/mod.md) | 配置加载器，分层加载 |
| [config_loader/macos.md](./config_loader/macos.md) | macOS MDM 托管配置 |

### 9️⃣ 上下文与历史管理

| 文档 | 说明 |
|------|------|
| [context_manager/mod.md](./context_manager/mod.md) | 上下文管理器入口 |
| [context_manager/history.md](./context_manager/history.md) | ContextManager，历史记录管理 |
| [context_manager/normalize.md](./context_manager/normalize.md) | 历史规范化 |
| [context_manager/truncate.md](./context_manager/truncate.md) | 输出截断和格式化 |
| [message_history.md](./message_history.md) | MessageHistory，消息历史 |
| [truncate.md](./truncate.md) | 文本截断工具函数 |

### 🔟 工具系统

#### 工具核心

| 文档 | 说明 |
|------|------|
| [tools/mod.rs.md](./tools/mod.rs.md) | 工具模块入口 |
| [tools/spec.rs.md](./tools/spec.rs.md) | 工具规范定义（JSON Schema） |
| [tools/registry.rs.md](./tools/registry.rs.md) | 工具注册表 |
| [tools/router.rs.md](./tools/router.rs.md) | 工具路由器 |
| [tools/orchestrator.rs.md](./tools/orchestrator.rs.md) | 工具编排器，审批和沙箱流程 |
| [tools/parallel.rs.md](./tools/parallel.rs.md) | 并行执行运行时 |
| [tools/context.rs.md](./tools/context.rs.md) | 工具上下文数据结构 |
| [tools/events.rs.md](./tools/events.rs.md) | 工具事件系统 |
| [tools/sandboxing.rs.md](./tools/sandboxing.rs.md) | 工具沙箱化接口 |

#### 工具处理器

| 文档 | 说明 |
|------|------|
| [tools/handlers/mod.md](./tools/handlers/mod.md) | 处理器模块入口 |
| [tools/handlers/apply_patch.md](./tools/handlers/apply_patch.md) | 应用补丁处理器 |
| [tools/handlers/grep_files.md](./tools/handlers/grep_files.md) | 文件搜索处理器 |
| [tools/handlers/list_dir.md](./tools/handlers/list_dir.md) | 目录列表处理器 |
| [tools/handlers/mcp.md](./tools/handlers/mcp.md) | MCP 工具调用处理器 |
| [tools/handlers/mcp_resource.md](./tools/handlers/mcp_resource.md) | MCP 资源处理器 |
| [tools/handlers/plan.md](./tools/handlers/plan.md) | 计划工具处理器 |
| [tools/handlers/read_file.md](./tools/handlers/read_file.md) | 读取文件处理器 |
| [tools/handlers/shell.md](./tools/handlers/shell.md) | Shell 命令处理器 |
| [tools/handlers/test_sync.md](./tools/handlers/test_sync.md) | 测试同步处理器 |
| [tools/handlers/unified_exec.md](./tools/handlers/unified_exec.md) | 统一执行处理器 |
| [tools/handlers/view_image.md](./tools/handlers/view_image.md) | 查看图像处理器 |

#### 执行运行时

| 文档 | 说明 |
|------|------|
| [tools/runtimes/mod.md](./tools/runtimes/mod.md) | 运行时模块入口 |
| [tools/runtimes/apply_patch.md](./tools/runtimes/apply_patch.md) | 补丁应用运行时 |
| [tools/runtimes/shell.md](./tools/runtimes/shell.md) | Shell 运行时 |
| [tools/runtimes/unified_exec.md](./tools/runtimes/unified_exec.md) | 统一执行运行时 |

### 1️⃣1️⃣ 命令执行与 Shell

| 文档 | 说明 |
|------|------|
| [exec.md](./exec.md) | 命令执行核心功能 |
| [exec_env.md](./exec_env.md) | 执行环境变量管理 |
| [bash.md](./bash.md) | Bash/Zsh 脚本解析 |
| [shell.md](./shell.md) | Shell 检测和配置 |
| [spawn.md](./spawn.md) | 进程生成功能 |
| [terminal.md](./terminal.md) | 终端模拟器检测 |

### 1️⃣2️⃣ 统一执行系统

| 文档 | 说明 |
|------|------|
| [unified_exec/mod.md](./unified_exec/mod.md) | 统一执行模块主入口 |
| [unified_exec/session.md](./unified_exec/session.md) | PTY 会话生命周期管理 |
| [unified_exec/session_manager.md](./unified_exec/session_manager.md) | 会话管理器核心逻辑 |
| [unified_exec/errors.md](./unified_exec/errors.md) | 统一执行错误类型 |

### 1️⃣3️⃣ 安全与沙箱

#### 命令安全

| 文档 | 说明 |
|------|------|
| [command_safety/mod.md](./command_safety/mod.md) | 命令安全模块入口 |
| [command_safety/is_safe_command.md](./command_safety/is_safe_command.md) | 安全命令白名单 |
| [command_safety/is_dangerous_command.md](./command_safety/is_dangerous_command.md) | 危险命令检测 |
| [command_safety/windows_safe_commands.md](./command_safety/windows_safe_commands.md) | Windows PowerShell 命令安全 |

#### 沙箱系统

| 文档 | 说明 |
|------|------|
| [sandboxing/mod.md](./sandboxing/mod.md) | 沙箱管理器核心 |
| [sandboxing/assessment.md](./sandboxing/assessment.md) | AI 驱动的风险评估 |
| [landlock.md](./landlock.md) | Linux Landlock + seccomp 沙箱 |
| [seatbelt.md](./seatbelt.md) | macOS Seatbelt 沙箱 |
| [safety.md](./safety.md) | 补丁安全性评估 |

### 1️⃣4️⃣ MCP（Model Context Protocol）

| 文档 | 说明 |
|------|------|
| [mcp/mod.md](./mcp/mod.md) | MCP 模块入口 |
| [mcp/auth.md](./mcp/auth.md) | MCP 认证机制 |
| [mcp_connection_manager.md](./mcp_connection_manager.md) | MCP 连接管理 |
| [mcp_tool_call.md](./mcp_tool_call.md) | MCP 工具调用 |

### 1️⃣5️⃣ 存储与会话记录

| 文档 | 说明 |
|------|------|
| [rollout/mod.md](./rollout/mod.md) | Rollout 模块入口 |
| [rollout/recorder.md](./rollout/recorder.md) | RolloutRecorder，会话记录器 |
| [rollout/list.md](./rollout/list.md) | 会话列表和分页 |
| [rollout/policy.md](./rollout/policy.md) | Rollout 策略 |

### 1️⃣6️⃣ 工具与实用功能

| 文档 | 说明 |
|------|------|
| [apply_patch.md](./apply_patch.md) | 补丁应用功能 |
| [function_tool.md](./function_tool.md) | 工具错误类型 |
| [custom_prompts.md](./custom_prompts.md) | 自定义提示 |
| [parse_command.md](./parse_command.md) | 命令解析 |
| [user_instructions.md](./user_instructions.md) | 用户指令处理 |
| [user_notification.md](./user_notification.md) | 用户通知 |
| [turn_diff_tracker.md](./turn_diff_tracker.md) | 轮次差异跟踪 |
| [util.md](./util.md) | 工具函数集合 |

### 1️⃣7️⃣ 项目与 Git 集成

| 文档 | 说明 |
|------|------|
| [git_info.md](./git_info.md) | Git 信息收集 |
| [project_doc.md](./project_doc.md) | 项目文档加载 |

### 1️⃣8️⃣ 环境与特性

| 文档 | 说明 |
|------|------|
| [environment_context.md](./environment_context.md) | 环境上下文信息 |
| [features.md](./features.md) | 功能特性系统 |
| [features/legacy.md](./features/legacy.md) | 遗留特性管理 |
| [flags.md](./flags.md) | 环境标志变量 |

### 1️⃣9️⃣ 格式化与映射

| 文档 | 说明 |
|------|------|
| [review_format.md](./review_format.md) | 审查结果格式化 |
| [event_mapping.md](./event_mapping.md) | 事件映射 |

### 2️⃣0️⃣ 遥测与监控

| 文档 | 说明 |
|------|------|
| [otel_init.md](./otel_init.md) | OpenTelemetry 初始化 |

### 2️⃣1️⃣ 错误处理

| 文档 | 说明 |
|------|------|
| [error.rs.md](./error.rs.md) | 错误类型系统 |

---

## 🔄 模块依赖关系

```
用户交互
    ↓
会话管理 (ConversationManager, CodexConversation)
    ↓
任务系统 (SessionTask: Regular, Compact, Review, Undo)
    ↓
    ├─→ AI 客户端 (ModelClient) → AI API
    ↓
工具系统 (ToolRouter → Orchestrator → Handlers → Runtimes)
    ↓
    ├─→ 沙箱检查 (SandboxingManager)
    ├─→ 命令安全 (CommandSafety)
    ├─→ 审批流程
    ↓
执行环境 (ExecEnv, UnifiedExec, PTY)
    ↓
    ├─→ 平台沙箱 (Landlock/Seatbelt/Windows)
    ├─→ 进程生成 (Spawn)
    ↓
存储层 (RolloutRecorder, MessageHistory)
    ↓
文件系统
```

---

## 🎯 快速开始

### 理解核心流程

1. **会话创建**: 从 `ConversationManager` 开始
2. **任务执行**: 查看 `tasks/regular.md`
3. **工具调用**: 阅读 `tools/router.md` 和 `tools/orchestrator.md`
4. **安全沙箱**: 了解 `sandboxing/mod.md`
5. **配置管理**: 参考 `config/mod.md`

### 添加新功能

- **新工具**: `tools/handlers/` + `tools/spec.rs`
- **新模型**: `model_provider_info.rs`
- **新任务**: `tasks/` + 实现 `SessionTask`
- **新沙箱**: `sandboxing/` + 平台特定实现

---

## 📊 文档统计

- **总文档数**: 110+ 个 Markdown 文件
- **覆盖模块**: 21 个主要模块
- **代码文件**: 100+ 个 Rust 源文件
- **语言**: 简体中文
- **更新时间**: 2025-11-07

---

## 📖 文档规范

每个文档包含：

1. **文件标题** - 清晰标识文件路径
2. **文件作用说明** - 简洁的 1-2 段描述
3. **主要结构体/枚举** - 关键数据类型及其字段
4. **主要函数/方法** - 核心功能及其参数
5. **与其他模块的关系** - 依赖和协作说明
6. **使用示例** - 实际代码示例（适用时）
7. **设计特点** - 关键设计决策（适用时）

---

## 🤝 贡献指南

如需更新文档：

1. 修改对应的 `.md` 文件
2. 确保使用简体中文
3. 保持简洁明了的风格
4. 更新本索引（如添加新模块）

---

## 📞 获取帮助

- 查看 [ARCHITECTURE.md](./ARCHITECTURE.md) 了解整体设计
- 搜索本文档查找特定模块
- 参考源代码中的内联注释

---

**文档维护**: Codex Team
**最后更新**: 2025-11-07
