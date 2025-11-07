# handlers/test_sync.rs

## 文件作用

实现测试同步工具处理器，提供测试中的延迟和同步屏障功能，主要用于测试并发和时序相关的场景。

## 主要结构体

### `TestSyncHandler`

测试同步工具的处理器，实现 `ToolHandler` trait。

**注意**：这是一个测试辅助工具，不应在生产环境中使用。

### `TestSyncArgs`

测试同步参数：
- `sleep_before_ms: Option<u64>` - 屏障前的延迟（毫秒）
- `sleep_after_ms: Option<u64>` - 屏障后的延迟（毫秒）
- `barrier: Option<BarrierArgs>` - 同步屏障配置

### `BarrierArgs`

同步屏障参数：
- `id: String` - 屏障 ID（用于识别同一屏障）
- `participants: usize` - 参与者数量
- `timeout_ms: u64` - 等待超时时间（默认 1000ms）

### `BarrierState`（内部）

屏障状态：
- `barrier: Arc<Barrier>` - Tokio 屏障实例
- `participants: usize` - 参与者数量

## 常量

- `DEFAULT_TIMEOUT_MS: u64 = 1000` - 默认超时时间（1 秒）

## 主要函数

### `handle()`

处理测试同步请求：
1. 如果指定了 `sleep_before_ms`，先等待
2. 如果指定了 `barrier`，调用 `wait_on_barrier()`
3. 如果指定了 `sleep_after_ms`，再等待
4. 返回 "ok"

### `wait_on_barrier()`

实现同步屏障等待：

**验证**：
1. 参与者数量必须大于 0
2. 超时时间必须大于 0

**屏障管理**：
1. 从全局映射中查找或创建屏障
2. 如果屏障已存在，验证参与者数量一致
3. 使用 Tokio `Barrier` 实现同步

**等待流程**：
1. 等待所有参与者到达屏障（带超时）
2. 如果是领导者（第一个到达），清理屏障
3. 超时则返回错误

## 全局状态

### `BARRIERS`

类型：`OnceLock<Mutex<HashMap<String, BarrierState>>>`

全局屏障映射，用于在多个调用间共享屏障实例。

## 使用示例

### 简单延迟

```json
{
  "sleep_before_ms": 100
}
```

### 同步屏障

```json
{
  "barrier": {
    "id": "test-barrier-1",
    "participants": 3,
    "timeout_ms": 5000
  }
}
```

等待 3 个参与者都到达 "test-barrier-1" 屏障后继续。

### 组合使用

```json
{
  "sleep_before_ms": 100,
  "barrier": {
    "id": "checkpoint",
    "participants": 2,
    "timeout_ms": 2000
  },
  "sleep_after_ms": 50
}
```

## 错误处理

- 参与者数量为 0：返回 "barrier participants must be greater than zero"
- 超时时间为 0：返回 "barrier timeout must be greater than zero"
- 参与者数量不一致：返回 "barrier {id} already registered with {existing} participants"
- 等待超时：返回 "test_sync_tool barrier wait timed out"

## 屏障生命周期

1. **创建**：第一个参与者到达时创建屏障
2. **等待**：所有参与者调用 `barrier.wait()`
3. **释放**：所有参与者到达后，领导者清理屏障
4. **超时**：如果超时，屏障保持在映射中（可能导致问题）

## 测试场景

### 并发工具调用测试

```rust
// 线程 1
test_sync_tool({
  "barrier": { "id": "start", "participants": 2 }
})

// 线程 2
test_sync_tool({
  "barrier": { "id": "start", "participants": 2 }
})
```

确保两个线程同时开始执行。

### 时序控制测试

```rust
// 第一步：延迟后同步
test_sync_tool({
  "sleep_before_ms": 100,
  "barrier": { "id": "phase1", "participants": 3 }
})

// 第二步：同步后延迟
test_sync_tool({
  "barrier": { "id": "phase2", "participants": 3 },
  "sleep_after_ms": 200
})
```

## 与其他模块的关系

- **依赖模块**：
  - `tokio::sync::Barrier` - 异步屏障实现
  - `tokio::time::sleep` - 异步延迟

- **使用场景**：
  - 测试工具并发执行
  - 测试时序相关的 bug
  - 验证同步机制
  - 模拟网络延迟

## 注意事项

- **仅用于测试**：不应在生产代码中启用此工具
- **内存泄漏风险**：超时的屏障不会被清理
- **参数一致性**：所有参与者必须使用相同的参与者数量
- **超时设置**：应根据测试环境设置合理的超时时间
