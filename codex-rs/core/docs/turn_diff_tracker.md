# turn_diff_tracker.rs

## 文件作用

`turn_diff_tracker.rs` 跟踪轮次之间的文件变更差异，用于生成轮次摘要。

## 主要结构体

### TurnDiffTracker
轮次差异跟踪器。

**方法:**
- `new()` - 创建跟踪器
- `record_change()` - 记录文件变更
- `get_diff()` - 获取差异摘要
- `clear()` - 清除跟踪数据

## 主要功能

- 跟踪文件创建、修改、删除
- 生成差异统计
- 支持增量更新

## 与其他模块的关系

**依赖模块:**
- `codex_protocol::protocol::TurnDiffEvent` - 差异事件类型

**被依赖模块:**
- `codex::Session` - 生成轮次差异事件
- `tools` - 记录工具操作的文件变更
