# context_manager/truncate.rs

## 文件作用

提供输出截断和格式化功能，控制发送给模型的内容大小，避免超出上下文窗口限制。

## 常量

- `MODEL_FORMAT_MAX_BYTES: usize = 10 * 1024` - 最大字节数（10 KiB）
- `MODEL_FORMAT_MAX_LINES: usize = 256` - 最大行数
- `MODEL_FORMAT_HEAD_LINES: usize = 128` - 头部保留行数
- `MODEL_FORMAT_TAIL_LINES: usize = 128` - 尾部保留行数
- `MODEL_FORMAT_HEAD_BYTES: usize = 5 * 1024` - 头部保留字节数（5 KiB）

## 主要函数

### globally_truncate_function_output_items

全局截断函数输出内容项列表。

**参数：**

- `items: &[FunctionCallOutputContentItem]` - 输出内容项列表

**返回：**

- `Vec<FunctionCallOutputContentItem>` - 截断后的内容项

**行为：**

- 维护 `MODEL_FORMAT_MAX_BYTES` 预算
- 按顺序处理内容项：
  - **InputText**: 截断或完整保留，扣除预算
  - **InputImage**: 完整保留（TODO: 未来考虑调整大小）
- 预算用尽后，统计省略的文本项数量
- 添加省略提示：`[omitted N text items ...]`

**字符边界：**

使用 `take_bytes_at_char_boundary` 确保不在 UTF-8 字符中间截断。

### format_output_for_model_body

格式化执行输出用于模型输入。

**参数：**

- `content: &str` - 原始输出内容

**返回：**

- `String` - 格式化后的输出

**行为：**

- 检查是否超过限制（字节和行数）
- 未超过：直接返回原文
- 超过：
  - 调用 `truncate_formatted_exec_output` 截断
  - 添加总行数前缀：`Total output lines: N`

### truncate_formatted_exec_output（私有）

执行头尾截断策略。

**截断策略：**

1. 保留前 128 行（头部）
2. 保留后 128 行（尾部）
3. 中间省略部分显示：`[... omitted N of M lines ...]`

**字节限制：**

- 头部最多 5 KiB
- 尾部使用剩余预算
- 使用 `take_bytes_at_char_boundary` 和 `take_last_bytes_at_char_boundary`

**省略标记：**

- 按行省略：`[... omitted N of M lines ...]`
- 按字节截断：`[... output truncated to fit 10240 bytes ...]`

## 辅助功能

**字符边界处理：**

- 依赖 `codex_utils_string` 的边界安全截断函数
- 确保 UTF-8 字符完整性

## 与其他模块的关系

- 被 `ContextManager::process_item` 调用，处理工具输出
- 使用 `codex_utils_string` 的字符边界处理函数
- 确保发送给模型的输出不超过限制
- 支持客户端接收完整流式输出，仅截断发送给模型的内容
- 平衡信息完整性和令牌效率：
  - 保留头尾信息（通常最重要）
  - 省略中间冗余内容
  - 为长输出提供摘要
