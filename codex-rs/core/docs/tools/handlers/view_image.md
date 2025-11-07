# handlers/view_image.rs

## 文件作用

实现图像查看工具处理器，允许 AI 模型将本地图像附加到当前对话上下文中，以便进行视觉分析。

## 主要结构体

### `ViewImageHandler`

图像查看工具的处理器，实现 `ToolHandler` trait。

### `ViewImageArgs`

图像查看参数：
- `path: String` - 图像文件路径（相对或绝对路径）

## 主要函数

### `handle()`

处理图像查看请求：

**处理流程**：
1. **解析参数**：
   - 提取 Function 载荷中的 arguments
   - 反序列化为 `ViewImageArgs`

2. **路径解析**：
   - 使用 `turn.resolve_path()` 解析为绝对路径
   - 支持相对路径（相对于当前工作目录）

3. **验证文件**：
   - 检查文件是否存在（`fs::metadata()`）
   - 验证路径是否指向文件（而非目录）

4. **注入图像**：
   - 调用 `session.inject_input()` 将图像添加到会话
   - 使用 `UserInput::LocalImage { path }` 类型

5. **发送事件**：
   - 发送 `ViewImageToolCallEvent` 通知客户端
   - 包含 call_id 和图像路径

6. **返回结果**：
   - 返回 "attached local image path" 消息

## 工作流程

```
AI 模型
  ↓ 调用 view_image(path: "screenshot.png")
ViewImageHandler
  ↓ 解析路径 → /workspace/screenshot.png
验证文件
  ↓ 文件存在且有效
Session::inject_input()
  ↓ 添加到会话上下文
发送 ViewImageToolCallEvent
  ↓ 通知客户端
返回成功
  ↓ "attached local image path"
AI 模型
  ↓ 可以在后续对话中引用和分析图像
```

## 错误处理

### 不支持的载荷

```
view_image handler received unsupported payload
```

### 参数解析失败

```
failed to parse function arguments: {error}
```

### 文件不存在

```
unable to locate image at `{path}`: {error}
```

### 路径不是文件

```
image path `{path}` is not a file
```

### 注入失败

```
unable to attach image (no active task)
```

当没有活跃的任务上下文时发生。

## 支持的图像格式

取决于底层的图像处理能力，通常支持：
- PNG
- JPEG / JPG
- GIF
- BMP
- WebP
- 其他常见格式

## 使用示例

### 分析截图

```json
{
  "path": "./screenshots/error-message.png"
}
```

AI 模型可以看到图像并分析错误信息。

### 检查 UI 设计

```json
{
  "path": "/home/user/designs/mockup-v2.png"
}
```

AI 模型可以审查设计并提供反馈。

### 调试可视化问题

```json
{
  "path": "test-output/plot.png"
}
```

AI 模型可以查看生成的图表并识别问题。

## 与其他模块的关系

- **依赖模块**：
  - `Session::inject_input()` - 注入用户输入
  - `tokio::fs` - 异步文件系统操作
  - `UserInput::LocalImage` - 图像输入类型

- **事件发送**：
  - `ViewImageToolCallEvent` - 图像查看事件
    - `call_id: String` - 工具调用 ID
    - `path: PathBuf` - 图像绝对路径

- **会话集成**：
  - 图像被添加到会话的输入流中
  - AI 模型在后续回复中可以引用图像
  - 多模态对话支持

## 安全考虑

- **路径验证**：确保路径指向实际文件
- **相对路径**：相对于工作目录解析，防止路径遍历
- **会话检查**：确保有活跃的任务才能注入

## 多模态能力

此工具使 AI 模型具备视觉能力：

- **代码理解**：查看架构图、流程图
- **调试辅助**：分析错误截图、崩溃报告
- **设计审查**：评估 UI/UX 设计
- **文档生成**：基于图像内容编写文档
- **数据分析**：查看图表、可视化结果

## 限制

- **本地文件**：仅支持本地文件系统上的图像
- **大小限制**：可能受模型的图像处理能力限制
- **同步性**：图像立即注入，无法撤销

## 设计特点

- **简单性**：单一职责，只负责附加图像
- **集成性**：无缝集成到会话上下文
- **事件驱动**：通过事件通知客户端
- **灵活性**：支持相对和绝对路径
