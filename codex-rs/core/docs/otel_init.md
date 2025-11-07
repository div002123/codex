# otel_init.rs

## 文件作用

`otel_init.rs` 初始化 OpenTelemetry 追踪和度量，用于性能监控和调试。

## 主要函数

### init_otel()
初始化 OpenTelemetry。

**功能:**
- 配置 tracing
- 配置 metrics
- 设置导出器
- 配置采样策略

## 与其他模块的关系

**依赖模块:**
- `codex_otel` - OpenTelemetry 工具库

**被依赖模块:**
- `main` - 程序启动时初始化
