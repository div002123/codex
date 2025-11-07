# Codex 项目构建运行指南

## 系统要求

| 项目 | 要求 |
|------|------|
| 操作系统 | macOS 12+、Ubuntu 20.04+/Debian 10+ 或 Windows 11 (WSL2) |
| Git | 2.23+ |
| RAM | 最低 4GB（推荐 8GB）|
| Node.js | 22+ |
| pnpm | 10.8.1+ |
| Rust | 最新稳定版 |

## 环境准备

### 1. 安装 Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt
rustup component add clippy
```

### 2. 安装 Node.js 22+

```bash
# 使用 nvm（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 22
nvm use 22

# 或使用系统包管理器
# Ubuntu/Debian:
# curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
# sudo apt-get install -y nodejs
```

### 3. 安装 pnpm

```bash
# 方法 1: 使用 corepack（Node.js 22+ 自带）
corepack enable
corepack prepare pnpm@10.8.1 --activate

# 方法 2: 使用 npm 全局安装
npm install -g pnpm@10.8.1
```

## 克隆项目

```bash
git clone https://github.com/openai/codex.git
cd codex
```

## 构建步骤

### 方案一：完整构建（Rust + TypeScript）

#### 1. 安装依赖

```bash
# 安装 Node.js 依赖（根目录和所有子包）
pnpm install
```

#### 2. 构建 Rust 项目

```bash
cd codex-rs
cargo build --release
```

#### 3. 构建 TypeScript SDK

```bash
cd ../sdk/typescript
pnpm run build
```

### 方案二：仅构建 Rust 项目

```bash
cd codex-rs
cargo build --release
```

### 方案三：开发模式构建

```bash
cd codex-rs
cargo build
```

## 运行

### 运行 Codex CLI

```bash
# 开发模式运行
cd codex-rs
cargo run --bin codex

# 带参数运行
cargo run --bin codex -- "explain this codebase to me"

# 使用编译后的二进制文件
./target/release/codex
```

### 运行 TypeScript SDK 示例

```bash
cd sdk/typescript
pnpm run build
node samples/example.js  # 如果有示例文件
```

## 测试

### Rust 测试

```bash
cd codex-rs

# 运行所有测试
cargo test

# 运行特定测试
cargo test --package codex-core

# 运行测试并显示输出
cargo test -- --nocapture
```

### TypeScript 测试

```bash
cd sdk/typescript

# 运行测试
pnpm test

# 运行测试（监听模式）
pnpm test:watch

# 运行代码覆盖率
pnpm coverage
```

## 代码质量检查

### Rust Lint 和格式化

```bash
cd codex-rs

# 格式化代码
cargo fmt -- --config imports_granularity=Item

# 运行 clippy 检查
cargo clippy --tests

# 自动修复 clippy 问题
cargo clippy --fix --tests
```

### TypeScript Lint 和格式化

```bash
# 根目录格式化
pnpm run format        # 检查
pnpm run format:fix    # 修复

# TypeScript SDK
cd sdk/typescript
pnpm run lint          # 检查
pnpm run lint:fix      # 修复
pnpm run format        # 检查格式
pnpm run format:fix    # 修复格式
```

## 开发工作流

### 1. 修改代码后的完整检查

```bash
cd codex-rs

# 格式化
cargo fmt -- --config imports_granularity=Item

# Lint 检查
cargo clippy --tests

# 运行测试
cargo test

# 构建发布版本
cargo build --release
```

### 2. 监听模式开发

```bash
# Rust: 使用 cargo-watch（需先安装）
cargo install cargo-watch
cargo watch -x 'run --bin codex'

# TypeScript SDK
cd sdk/typescript
pnpm run build:watch
```

### 3. 工作空间命令（pnpm）

```bash
# 在特定包中运行命令
pnpm --filter @openai/codex-sdk run build

# 在所有包中运行命令
pnpm -r run test

# 为特定包添加依赖
pnpm --filter @openai/codex-sdk add lodash
```

## 常见问题

### 清理构建产物

```bash
# Rust
cd codex-rs
cargo clean

# TypeScript
cd sdk/typescript
pnpm run clean
rm -rf node_modules

# 完整清理（根目录）
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### 更新依赖

```bash
# 更新 Rust 依赖
cd codex-rs
cargo update

# 更新 pnpm 依赖
pnpm update
```

### 构建失败排查

```bash
# 1. 确认 Rust 版本
rustc --version

# 2. 确认 Node.js 和 pnpm 版本
node --version
pnpm --version

# 3. 清理并重新构建
cd codex-rs
cargo clean
cargo build

# 4. 重新安装 Node.js 依赖
cd ../
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

## 生产部署

### 构建优化版本

```bash
cd codex-rs
cargo build --release

# 可执行文件位置
ls -lh target/release/codex
```

### 安装到系统

```bash
# 从源码安装
cd codex-rs
cargo install --path cli

# 验证安装
codex --version
```

## 项目结构说明

```
codex/
├── codex-rs/           # Rust 工作空间（主要代码）
│   ├── cli/            # CLI 入口
│   ├── core/           # 核心逻辑
│   ├── tui/            # 终端界面
│   ├── app-server/     # 应用服务器
│   └── ...             # 其他模块
├── sdk/                # SDK
│   └── typescript/     # TypeScript SDK
├── docs/               # 文档
├── scripts/            # 脚本工具
├── package.json        # 根 package.json
└── pnpm-workspace.yaml # pnpm 工作空间配置
```

## 贡献代码前检查清单

```bash
# 1. 格式化代码
cargo fmt -- --config imports_granularity=Item

# 2. Lint 检查
cargo clippy --tests

# 3. 运行测试
cargo test

# 4. 确保所有测试通过
echo "✓ 所有检查通过，可以提交代码"
```
