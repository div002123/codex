# exec_env.rs

## 文件作用

根据 `ShellEnvironmentPolicy` 创建执行环境的环境变量映射。

该模块实现了一个复杂的环境变量派生算法，支持继承策略、默认排除、自定义排除、变量设置和包含过滤。

## 主要函数

### create_env
```rust
pub fn create_env(policy: &ShellEnvironmentPolicy) -> HashMap<String, String>
```

根据策略创建环境变量映射。

使用当前进程的环境变量作为输入，应用策略规则。

### populate_env
```rust
fn populate_env<I>(vars: I, policy: &ShellEnvironmentPolicy) -> HashMap<String, String>
where
    I: IntoIterator<Item = (String, String)>
```

从给定的变量迭代器派生环境映射（用于测试）。

## 环境变量派生算法

按以下顺序应用 5 个步骤：

### 步骤 1：确定起始变量集
根据 `inherit` 策略：

**ShellEnvironmentPolicyInherit::All**
- 继承所有父进程的环境变量

**ShellEnvironmentPolicyInherit::None**
- 从空映射开始

**ShellEnvironmentPolicyInherit::Core**
- 只继承核心变量：
  - `HOME`
  - `LOGNAME`
  - `PATH`
  - `SHELL`
  - `USER`
  - `USERNAME`
  - `TMPDIR`
  - `TEMP`
  - `TMP`

### 步骤 2：应用默认排除
如果 `ignore_default_excludes` 为 `false`，排除包含以下模式的变量（不区分大小写）：
- `*KEY*`
- `*SECRET*`
- `*TOKEN*`

这些模式旨在防止意外泄露敏感信息。

### 步骤 3：应用自定义排除
从映射中删除匹配 `policy.exclude` 中任何模式的变量。

### 步骤 4：应用用户提供的覆盖
将 `policy.set` 中的所有键值对插入映射，覆盖现有值。

### 步骤 5：应用包含过滤
如果 `policy.include_only` 非空，只保留匹配其中任何模式的变量。

## 辅助函数

### matches_any
```rust
let matches_any = |name: &str, patterns: &[EnvironmentVariablePattern]| -> bool {
    patterns.iter().any(|pattern| pattern.matches(name))
}
```

检查变量名是否匹配任何给定的模式。

## 使用示例

### 示例 1：默认策略（Core 继承 + 默认排除）
```rust
let policy = ShellEnvironmentPolicy::default();
let env = create_env(&policy);
// 结果：只包含 PATH, HOME 等核心变量，排除 API_KEY, SECRET_TOKEN 等
```

### 示例 2：只包含特定变量
```rust
let policy = ShellEnvironmentPolicy {
    ignore_default_excludes: true,
    include_only: vec![EnvironmentVariablePattern::new_case_insensitive("*PATH")],
    ..Default::default()
};
let env = populate_env(vars, &policy);
// 结果：只包含匹配 *PATH 的变量
```

### 示例 3：设置新变量
```rust
let mut policy = ShellEnvironmentPolicy {
    ignore_default_excludes: true,
    ..Default::default()
};
policy.set.insert("NEW_VAR".to_string(), "42".to_string());
let env = populate_env(vars, &policy);
// 结果：包含核心变量 + NEW_VAR
```

### 示例 4：继承所有变量
```rust
let policy = ShellEnvironmentPolicy {
    inherit: ShellEnvironmentPolicyInherit::All,
    ignore_default_excludes: true,
    ..Default::default()
};
let env = populate_env(vars, &policy);
// 结果：所有父进程的环境变量
```

### 示例 5：无继承，只设置特定变量
```rust
let mut policy = ShellEnvironmentPolicy {
    inherit: ShellEnvironmentPolicyInherit::None,
    ignore_default_excludes: true,
    ..Default::default()
};
policy.set.insert("ONLY_VAR".to_string(), "yes".to_string());
let env = populate_env(vars, &policy);
// 结果：只包含 ONLY_VAR
```

## 与其他模块的关系

- **unified_exec/session_manager.rs**：使用 `create_env` 创建执行环境
- **sandboxing**：执行环境的环境变量由此模块创建
- **config/types.rs**：使用 `ShellEnvironmentPolicy` 和相关类型
- **spawn.rs**：生成的子进程使用此模块创建的环境变量映射
