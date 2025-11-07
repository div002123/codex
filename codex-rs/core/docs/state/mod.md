# state/mod.rs

## 文件作用

`state/mod.rs` 是 state 模块的入口文件，负责组织和导出模块内的各个子模块。

## 主要导出

### 公开导出 (pub(crate))

- **`SessionServices`**: 从 `service.rs` 导出，管理会话级别的服务依赖
- **`SessionState`**: 从 `session.rs` 导出，管理会话级别的可变状态
- **`ActiveTurn`**: 从 `turn.rs` 导出，管理当前活跃轮次的元数据
- **`RunningTask`**: 从 `turn.rs` 导出，表示正在运行的任务
- **`TaskKind`**: 从 `turn.rs` 导出，任务类型枚举

## 模块结构

该模块包含三个子模块：
- `service`: 会话服务管理
- `session`: 会话状态管理
- `turn`: 轮次状态管理

## 与其他模块的关系

- 被 `codex::Session` 模块使用，提供会话和轮次的状态管理
- 与 `tasks` 模块协作，定义任务类型和运行时状态
- 依赖 `context_manager` 模块管理对话历史
