# Codex Core Standalone

这是从 `codex-rs/core` 模块提取出来的独立 Rust 项目。该项目包含了所有必要的依赖，可以独立构建和运行。

## 项目结构

```
codex-core-standalone/
├── core/                    # 核心模块 (codex-core)
├── mcp-types/              # MCP 类型定义
├── protocol/               # 协议定义
├── app-server-protocol/    # 应用服务协议
├── apply-patch/            # 补丁应用工具
├── async-utils/            # 异步工具库
├── file-search/            # 文件搜索功能
├── keyring-store/          # 密钥存储
├── otel/                   # OpenTelemetry 集成
├── rmcp-client/            # RMCP 客户端
├── windows-sandbox-rs/     # Windows 沙箱（仅Windows平台）
├── utils/                  # 工具库集合
│   ├── cache/             # 缓存工具
│   ├── git/               # Git 工具
│   ├── image/             # 图像处理
│   ├── pty/               # PTY 工具
│   ├── readiness/         # 就绪检查
│   ├── string/            # 字符串工具
│   └── tokenizer/         # 分词器
├── Cargo.toml             # 工作区配置
└── Cargo.lock             # 依赖锁定文件
```

## 构建

### 构建所有库

```bash
cargo build --lib
```

### 构建特定的 crate

```bash
# 构建 core 库
cargo build --lib -p codex-core

# 构建其他 crate
cargo build --lib -p codex-protocol
cargo build --lib -p mcp-types
```

### 运行检查

```bash
cargo check --lib
```

## 依赖说明

### 内部依赖 (Workspace)

- **core**: 核心库 - 包含 Codex 的主要功能
- **protocol**: 通信协议定义
- **app-server-protocol**: 应用服务器协议
- **mcp-types**: MCP (Model Context Protocol) 类型定义
- **Utils**: 各种工具库（git、缓存、搜索等）

### 外部依赖

所有外部依赖均来自 crates.io，主要包括：
- **tokio**: 异步运行时
- **serde/serde_json**: 序列化库
- **reqwest**: HTTP 客户端
- **tree-sitter**: 代码解析
- **opentelemetry**: 遥测系统
- 以及其他 100+ 个依赖库

## 修改依赖

### 添加新的内部依赖

如果需要添加新的 crate，请：

1. 将 crate 复制到项目中
2. 在 `Cargo.toml` 的 `[workspace]` 部分的 `members` 列表中添加它
3. 在 `[workspace.dependencies]` 部分添加对应的路径依赖

### 添加新的外部依赖

在 `Cargo.toml` 的 `[workspace.dependencies]` 部分添加新的依赖即可。

## 平台特定配置

某些依赖是平台特定的：

- **Linux**: `landlock`, `seccompiler` (沙箱支持)
- **macOS**: `core-foundation` (系统集成)
- **musl targets**: `openssl-sys` (OpenSSL 构建)
- **Windows**: `codex-windows-sandbox` (沙箱支持)

## 发源信息

本项目是从以下原始位置提取的：
- 源仓库: `https://github.com/anthropics/codex`
- 原始路径: `codex-rs/core` 及其依赖模块
- 提取时间: 2025-11-10

## 注意事项

1. **功能不变**: 代码功能与原始项目完全相同，仅是重新组织为独立项目
2. **依赖完整**: 所有必要的依赖都已复制，项目可以独立构建
3. **测试支持**: 包含测试支持库 (`core_test_support`)
4. **工作区结构**: 使用 Cargo 工作区管理多个 crate，便于依赖管理

## 使用场景

这个独立项目适用于：
- 学习和研究 Codex 核心模块的实现
- 独立开发和测试 core 模块的功能
- 将 core 模块集成到其他项目中
- 对 core 模块进行性能优化和改进

## 许可证

与原始 Codex 项目保持一致。
