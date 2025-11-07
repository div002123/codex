# features.rs

## 文件作用

`features.rs` 定义功能特性标志系统，用于控制实验性和可选功能的启用/禁用。

## 主要枚举

### Feature
功能特性枚举。

**变体:**
- `UnifiedExec` - 统一的 PTY 执行工具
- `RmcpClient` - RMCP 客户端（OAuth 登录等）
- `ApplyPatchFreeform` - 自由格式补丁工具
- `ViewImageTool` - 查看图片工具
- `WebSearchRequest` - Web 搜索请求
- `SandboxCommandAssessment` - 沙箱命令风险评估
- `GhostCommit` - Ghost 提交
- `WindowsSandbox` - Windows 沙箱

**方法:**
- `key()` - 特性键名
- `stage()` - 生命周期阶段
- `default_enabled()` - 默认是否启用

### Stage
功能生命周期阶段。

**变体:**
- `Experimental` - 实验性
- `Beta` - 测试版
- `Stable` - 稳定版
- `Deprecated` - 已弃用
- `Removed` - 已移除

## 主要结构体

### Features
功能特性集合。

**字段:**
- `enabled: BTreeSet<Feature>` - 已启用的特性
- `legacy_usages: BTreeSet<LegacyFeatureUsage>` - 遗留用法

**方法:**
- `is_enabled()` - 检查特性是否启用
- `enable()` - 启用特性
- `disable()` - 禁用特性

### FeatureOverrides
功能特性覆盖配置。

**字段:**
- `include_apply_patch_tool: Option<bool>` - 包含补丁工具
- `web_search_request: Option<bool>` - Web 搜索请求
- `experimental_sandbox_command_assessment: Option<bool>` - 沙箱命令评估

## 子模块

### legacy
遗留功能特性切换。

## 与其他模块的关系

**依赖模块:**
- `config::{ConfigToml, profile::ConfigProfile}` - 配置加载

**被依赖模块:**
- `codex::SessionConfiguration` - 会话配置中使用
- `tools::ToolsConfig` - 根据特性配置工具
- 各功能模块 - 检查特性是否启用

**核心流程:**
配置加载 → 解析特性标志 → 创建 Features 对象 → 各模块检查特性 → 启用/禁用功能
