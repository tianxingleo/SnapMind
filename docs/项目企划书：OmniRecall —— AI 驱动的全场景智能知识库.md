# 项目企划书：OmniRecall —— AI 驱动的全场景智能知识库

**版本号：** v1.0

**日期：** 2025-11-24

**技术栈：** Frontend (Vue 3) / Backend (Go) / Desktop Wrapper (Tauri 推荐)

## 1. 项目愿景 (Vision)

打造一款**“第二大脑”**级别的桌面端应用。它不只是一个截图工具，而是一个**能够感知屏幕上下文的智能采集器**。通过“截取即存储，存储即知识”的理念，将用户在屏幕上看到的一切（文本、手写体、公式、视频、网页、文件）转化为结构化的 Markdown 笔记，沉淀为本地向量知识库，并允许用户通过自然语言随时调用和回溯。

**核心标语：** Capture Everything, Recall Anything.

## 2. 核心功能模块 (Core Features)

### 2.1 智能触发与采集 (Smart Capture)

- **全局长按呼出：**
  - 监听键盘事件（如长按 `Ctrl` 或 `Alt`），触发屏幕冻结层（Overlay）。
- **智能吸附 (Auto-Snap)：**
  - 当鼠标悬停在某个窗口上时，自动高亮识别该窗口区域。
  - 支持手动框选区域（类似 Snipaste）。
- **深度上下文获取 (Context Extraction)：**
  - **浏览器场景：** 通过 Accessibility API 或浏览器插件，获取当前 Tab 的 URL、Title 以及 DOM 纯文本。
  - **文件管理器场景：** 如果框选的是文件图标或打开的文档窗口，自动获取文件路径，并读取文件内容（PDF/Word/TXT）。
  - **视频场景：** 识别当前是否为视频播放器（网页或本地），若为视频，提取当前帧画面 + 提取时间戳附近的字幕（若有）或音频片段。

### 2.2 多模态处理引擎 (Multimodal Processing Engine)

- **OCR 增强 (Pix2Text 集成)：**
  - 针对常规截图：提取文本。
  - 针对手写体/字迹不清：调用专用增强模型进行识别。
  - **数学公式：** 识别 LaTeX 格式的数学公式，便于 Markdown 渲染。
- **视频/图像分析：**
  - 对截取的视频帧进行画面描述（Image-to-Text）。
- **隐私确认机制 (Privacy Guard)：**
  - 在上传云端大模型分析前，若检测到敏感关键词（密码、身份证号等）或设计本地文件读取，弹出二次确认框，允许用户对敏感信息打码或取消上传。

### 2.3 LLM 智能处理与笔记生成 (AI Analysis & Note Taking)

- **信息精炼：**
  - 保存原始 OCR 数据。
  - 调用 LLM 总结核心观点。
  - 自动生成标签（Tags）和分类（Categories）。
- **Markdown 标准化：** 将所有来源（网页、图片、视频描述）统一转化为标准 Markdown 格式。
- **即时问询 (Ask on Capture)：** 截图完成后，弹窗允许用户直接针对当前截图内容向 AI 提问（例如：“解释这个公式的推导过程”）。

### 2.4 本地知识库与 RAG (Local Knowledge Base)

- **自动归档：** 根据 AI 生成的标签，自动归入相应的文件夹。
- **向量化存储：** 使用嵌入模型（Embedding Model）将笔记内容向量化，存入本地向量数据库。
- **知识库对话：**
  - 类似 NotebookLM 的体验。用户可以提问：“我上周看的关于 Vue3 的那个视频里讲了什么？”
  - 系统基于 RAG（检索增强生成）从本地历史记录中检索答案。

## 3. 技术架构 (Technical Architecture)

### 3.1 客户端架构 (Desktop Application)

- **框架选型：** **Tauri v2** (推荐) 或 Electron。
  - *理由：* 你选择了 Go 作为后端，Tauri 使用 Rust 构建核心但也非常适合与 Go 的 Sidecar 模式配合，或者直接用 Go 编写后端服务并通过 HTTP/gRPC 与 Vue 前端通信。Tauri 相比 Electron 更轻量，更符合“后台常驻工具”的定位。
- **前端 (Frontend)：**
  - **Vue 3 (Composition API) + TypeScript + Vite**。
  - **UI 库：** Naive UI 或 Tailwind CSS (追求轻量化)。
  - **功能：** 负责截图交互、Markdown 渲染、聊天界面、设置界面。

### 3.2 后端服务 (Backend Service - Go)

- **角色：** 本地常驻服务 (Daemon)，处理业务逻辑、文件 I/O、数据库操作。
- **框架：** Gin 或 Fiber。
- **核心模块：**
  - **System Hook：** 监听键盘长按事件 (使用 `github.com/go-vgo/robotgo` 或操作系统原生 API)。
  - **Window API：** 获取当前活动窗口句柄、进程名、URL (可能需要调用 OS 特定 API 或 Windows Automation API)。
  - **Data Pipeline：** 管理 OCR -> LLM -> Vector DB 的流水线。

### 3.3 AI 基础设施 (AI Layer)

鉴于 Go 在 AI 生态上的劣势，建议采用 **Go 调用 Python Microservice** 或 **直接调用 API** 的混合模式：

- **OCR/Pix2Text：** 建议封装一个轻量级的 Python RPC 服务（或使用 ONNX Runtime 在 Go 中直接加载模型，如果模型支持）。
- **LLM 接入：**
  - 云端：OpenAI API, Claude, Gemini, DeepSeek (通过 Go HTTP Client 调用)。
  - 本地：Ollama (Go 直接调用 Ollama 接口) 运行 Llama 3 或 Qwen 模型。
- **向量数据库：**
  - 嵌入式方案：SQLite (利用 sqlite-vec 插件) 或 ChromaDB (本地部署)。

## 4. 用户交互流程设计 (UX Flow)

1. **激活 (Trigger)：** 用户长按 `Caps Lock` 或 `Alt`。
2. **定格 (Freeze)：** 屏幕变暗，光标下的窗口高亮（显示应用名/URL）。
3. **选择 (Select)：** 用户点击确认，或拖拽修正区域。
4. **确认 (Confirm)：** 若涉及文件读取，右下角弹出 Toast："正在读取 [文件名]，是否继续？"。
5. **处理 (Processing)：**
   - 前端显示加载动画。
   - 后端并行处理 OCR 和 LLM 分析。
6. **结果 (Result)：**
   - 屏幕右侧滑出“速记卡片”。
   - 展示：原文、摘要、标签。
   - 操作：点击“保存”或输入框继续提问。
7. **沉淀 (Archive)：** 卡片飞入“知识库”图标，后台静默完成向量化。

## 5. 隐私与安全策略 (Privacy & Security)

- **本地优先 (Local First)：** 默认情况下，数据库、截图文件全部存储在用户本地硬盘。
- **敏感信息过滤 (PII Redaction)：** 在 OCR 识别出身份证、银行卡、手机号格式时，自动进行掩码处理 (`138****0000`)。
- **透明化传输：** 如果使用云端 LLM，必须在设置中明确告知用户哪些数据会被发送，并提供“仅使用本地 LLM (Ollama)”的选项。

## 6. 开发路线图 (Roadmap)

### Phase 1: 原型验证 (MVP)

- [ ] 实现 Vue3 + Go (Tauri) 的基础框架搭建。
- [ ] 实现全局长按快捷键呼出 Overlay。
- [ ] 实现基础截图 + 窗口位置自动识别 (Windows/Mac)。
- [ ] 接入通用 OCR API (如 Tesseract 或 百度/阿里 OCR API 先行测试)。
- [ ] 实现简单的 Markdown 预览与本地保存。

### Phase 2: 深度智能 (Intelligence)

- [ ] 集成 Pix2Text 模型（数学公式、手写体优化）。
- [ ] 接入浏览器上下文获取功能 (URL 提取)。
- [ ] 接入 LLM API 进行笔记总结和打标签。
- [ ] 开发“截屏即问”的对话界面。

### Phase 3: 知识库闭环 (Knowledge Base)

- [ ] 引入向量数据库 (RAG 基础)。
- [ ] 实现全局搜索（语义搜索）。
- [ ] 视频/文件内容的深度解析功能。
- [ ] UI/UX 精修，增加动画效果。

## 7. 竞品差异化分析 (Differentiation)

| 特性         | Windows Recall | NotebookLM      | Snipaste   | **OmniRecall (本项目)**         |
| ------------ | -------------- | --------------- | ---------- | ------------------------------- |
| **触发方式** | 后台全时段录制 | 文件导入        | 快捷键截图 | **按需触发 + 智能吸附**         |
| **数据源**   | 纯视觉回溯     | 上传的文档      | 纯图片     | **图片 + 链接 + 文件 + 视频流** |
| **处理深度** | 关键词检索     | 深度理解与问答  | 无         | **多模态理解 + 知识库构建**     |
| **隐私感**   | 低 (备受争议)  | 中 (上传Google) | 高         | **高 (本地控制 + 隐私确认)**    |

## 8. 总结

OmniRecall 旨在填补“快速采集”与“深度整理”之间的鸿沟。利用 Go 的高性能处理系统 I/O，Vue3 构建现代化的交互界面，以及 LLM 的认知能力，构建下一代个人知识管理工具。