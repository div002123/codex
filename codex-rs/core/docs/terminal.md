# terminal.rs

## 文件作用

检测用户使用的终端模拟器类型和版本，用于生成 User-Agent 字符串。

该模块通过检查环境变量来识别常见的终端模拟器，并将其格式化为合适的标识字符串。

## 全局状态

### TERMINAL
```rust
static TERMINAL: OnceLock<String> = OnceLock::new();
```

使用 `OnceLock` 存储检测到的终端类型，确保检测只执行一次。

## 主要函数

### user_agent
```rust
pub fn user_agent() -> String
```

获取终端标识字符串，用于 User-Agent。

**特点**：
- 懒惰初始化：首次调用时检测，后续调用直接返回缓存结果
- 线程安全：使用 `OnceLock` 保证多线程环境下只检测一次

### detect_terminal
```rust
fn detect_terminal() -> String
```

检测终端类型的核心逻辑。

**检测顺序**：
1. `TERM_PROGRAM` + `TERM_PROGRAM_VERSION`
2. `WEZTERM_VERSION`
3. `KITTY_WINDOW_ID` 或 `TERM=*kitty*`
4. `ALACRITTY_SOCKET` 或 `TERM=alacritty`
5. `KONSOLE_VERSION`
6. `GNOME_TERMINAL_SCREEN`
7. `VTE_VERSION`
8. `WT_SESSION`（Windows Terminal）
9. 回退到 `TERM` 变量
10. 最终回退：`"unknown"`

### sanitize_header_value
```rust
fn sanitize_header_value(value: String) -> String
```

清理终端名称，使其适合作为 HTTP 头部值。

**规则**：
- 保留：字母数字、`-`、`_`、`.`、`/`
- 替换：其他字符替换为 `_`

### is_valid_header_value_char
```rust
fn is_valid_header_value_char(c: char) -> bool
```

检查字符是否适合用于 HTTP 头部值。

## 支持的终端

### 通用（TERM_PROGRAM）
**环境变量**：`TERM_PROGRAM`, `TERM_PROGRAM_VERSION`

**支持的终端**：
- iTerm.app（macOS）
- Apple_Terminal（macOS）
- vscode（VS Code 内置终端）
- 等等

**格式**：
- 有版本：`iTerm.app/3.4.16`
- 无版本：`iTerm.app`

### WezTerm
**环境变量**：`WEZTERM_VERSION`

**格式**：
- 有版本：`WezTerm/20230408-112425-69ae8472`
- 无版本：`WezTerm`

### Kitty
**环境变量**：`KITTY_WINDOW_ID` 或 `TERM=xterm-kitty`

**格式**：`kitty`

### Alacritty
**环境变量**：`ALACRITTY_SOCKET` 或 `TERM=alacritty`

**格式**：`Alacritty`

### Konsole
**环境变量**：`KONSOLE_VERSION`

**格式**：
- 有版本：`Konsole/22.12.3`
- 无版本：`Konsole`

### GNOME Terminal
**环境变量**：`GNOME_TERMINAL_SCREEN`

**格式**：`gnome-terminal`

### VTE-based
**环境变量**：`VTE_VERSION`

**格式**：
- 有版本：`VTE/7200`
- 无版本：`VTE`

**基于 VTE 的终端**：
- GNOME Terminal
- XFCE Terminal
- Tilix
- 等等

### Windows Terminal
**环境变量**：`WT_SESSION`

**格式**：`WindowsTerminal`

### 回退
**环境变量**：`TERM`

**格式**：`TERM` 变量的值（如 `xterm-256color`, `screen-256color`）

## 使用示例

### 示例 1：获取终端标识
```rust
let terminal = user_agent();
println!("Terminal: {}", terminal);
// 可能的输出：
// - "iTerm.app/3.4.16"
// - "WezTerm/20230408-112425-69ae8472"
// - "kitty"
// - "Alacritty"
// - "WindowsTerminal"
// - "xterm-256color"
```

### 示例 2：在 HTTP 请求中使用
```rust
let user_agent = format!("Codex/1.0 ({})", user_agent());
// 结果：Codex/1.0 (iTerm.app/3.4.16)
```

### 示例 3：清理不安全字符
```rust
let raw = "My Terminal (v1.0)";
let sanitized = sanitize_header_value(raw.to_string());
// 结果：My_Terminal__v1.0_
```

## 检测逻辑详解

### iTerm2 示例
```rust
// 环境变量：
// TERM_PROGRAM=iTerm.app
// TERM_PROGRAM_VERSION=3.4.16
if let Ok(tp) = std::env::var("TERM_PROGRAM") && !tp.trim().is_empty() {
    let ver = std::env::var("TERM_PROGRAM_VERSION").ok();
    match ver {
        Some(v) if !v.trim().is_empty() => format!("{tp}/{v}"),
        _ => tp,
    }
}
// 结果：iTerm.app/3.4.16
```

### WezTerm 示例
```rust
// 环境变量：WEZTERM_VERSION=20230408-112425-69ae8472
if let Ok(v) = std::env::var("WEZTERM_VERSION") {
    if !v.trim().is_empty() {
        format!("WezTerm/{v}")
    } else {
        "WezTerm".to_string()
    }
}
// 结果：WezTerm/20230408-112425-69ae8472
```

### Kitty 示例
```rust
// 环境变量：KITTY_WINDOW_ID=1 或 TERM=xterm-kitty
if std::env::var("KITTY_WINDOW_ID").is_ok()
    || std::env::var("TERM")
        .map(|t| t.contains("kitty"))
        .unwrap_or(false)
{
    "kitty".to_string()
}
// 结果：kitty
```

## 特殊情况

### 空值处理
所有检测都会验证环境变量值不为空（`!v.trim().is_empty()`）。

### 多重检测
某些终端可能设置多个环境变量，检测按优先级顺序进行。

### 版本格式
不同终端的版本格式各异：
- iTerm2：`3.4.16`（语义化版本）
- WezTerm：`20230408-112425-69ae8472`（日期+提交哈希）
- Konsole：`22.12.3`（年.月.修订）

## 与其他模块的关系

- **http/client**：使用终端标识构建 User-Agent 字符串
- **config**：终端检测可能影响配置推荐
- **shell.rs**：配合使用，终端和 shell 通常相关
- **protocol**：User-Agent 可能包含在协议消息中
- **telemetry**：终端信息可能用于使用统计

## 注意事项

1. **缓存**：终端检测结果被缓存，运行期间不会改变
2. **环境变量**：依赖环境变量，在某些非交互式场景可能不准确
3. **清理**：所有值都经过清理，确保可以安全用于 HTTP 头部
4. **回退**：总是有合理的回退值（`"unknown"`）
5. **性能**：懒惰初始化，只在首次调用时检测
