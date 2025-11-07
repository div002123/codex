# features/legacy.rs

## 文件作用

`features/legacy.rs` 处理遗留的功能特性配置格式，提供向后兼容性。

## 主要结构体

### LegacyFeatureToggles
遗留功能切换。

**字段:**
- 各种遗留布尔标志（如 `include_apply_patch_tool`、`tools_web_search` 等）

**方法:**
- `apply()` - 应用到新的 Features 系统

## 与其他模块的关系

**依赖模块:**
- `features::{Feature, Features}` - 新的功能系统

**被依赖模块:**
- `config` - 加载遗留配置时使用
