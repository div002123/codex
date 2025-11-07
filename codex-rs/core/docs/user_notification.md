# user_notification.rs

## 文件作用

`user_notification.rs` 提供用户通知功能，用于发送后台事件和状态更新。

## 主要结构体

### UserNotifier
用户通知器。

**方法:**
- `notify()` - 发送通知
- `notify_background_event()` - 发送后台事件
- `notify_stream_error()` - 发送流错误通知

### UserNotification
用户通知消息。

**字段:**
- `message: String` - 通知消息
- `level: NotificationLevel` - 通知级别

### NotificationLevel
通知级别枚举。

**变体:**
- `Info` - 信息
- `Warning` - 警告
- `Error` - 错误

## 与其他模块的关系

**被依赖模块:**
- `codex::Session` - 发送各种通知
- `compact` - 发送压缩进度通知
