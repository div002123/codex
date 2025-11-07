# review_format.rs

## 文件作用

`review_format.rs` 格式化代码审查输出，生成友好的审查报告。

## 主要函数

### format_review()
格式化审查结果。

**参数:**
- `review_data: &ReviewData` - 审查数据

**返回:** `String` - 格式化后的审查报告

**功能:**
- 格式化问题列表
- 添加严重程度标记
- 生成摘要
- 美化输出

## 与其他模块的关系

**被依赖模块:**
- `tasks::ReviewTask` - 生成审查报告
