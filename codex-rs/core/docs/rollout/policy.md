# rollout/policy.rs

## 文件作用

`rollout/policy.rs` 定义 rollout 策略，控制哪些数据需要持久化。

## 主要枚举

### RolloutPolicy
Rollout 策略。

**变体:**
- `All` - 持久化所有数据
- `Essential` - 仅持久化必要数据
- `None` - 不持久化

## 与其他模块的关系

**被依赖模块:**
- `rollout::recorder` - 根据策略决定是否记录
