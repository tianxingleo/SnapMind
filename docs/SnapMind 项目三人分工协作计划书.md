# SnapMind 项目三人分工协作计划书

**项目名称：** SnapMind (瞬脑) **项目目标：** 构建一个基于截图入口、由 AI 驱动的本地化个人知识库闭环系统。 **技术架构：** Client-Server 微服务架构 (Python Client + Go Server + Vue Frontend)

## 1. 团队角色定义

| 成员代号   | 角色头衔              | 主要技术栈                | 核心职责                                                     |
| ---------- | --------------------- | ------------------------- | ------------------------------------------------------------ |
| **成员 A** | **感知层工程师**      | Python, PySide6/PyQt, OCR | **"眼睛与手"**。负责桌面交互、截图、本地 OCR 推理，保障信息采集的精度与速度。 |
| **成员 B** | **后端/AI 工程师**    | Go (Gin), SQL, Vector DB  | **"大脑与记忆"**。负责数据存储、API 接口、向量检索 (RAG) 逻辑、调度本地 LLM。 |
| **成员 C** | **前端/可视化工程师** | Vue 3, Tailwind, ECharts  | **"面孔与交互"**。负责知识呈现、Markdown/LaTeX 渲染、用户问答界面、知识图谱。 |

## 2. 详细任务分解与实现路径

### 👨‍💻 成员 A：桌面端 (Python Agent)

**核心任务：** 实现“无感截图”到“数据上报”的闭环。

#### 具体功能与实现方案

1. **系统托盘与生命周期管理：**
   - **实现：** 使用 `pystray` 或 `PyQt6` 实现系统托盘图标，程序启动后隐藏主窗口，后台常驻。
2. **全局快捷键与截图交互：**
   - **实现：** 使用 `keyboard` 库监听 `Alt+A`。触发后，使用 `PyQt6` 绘制全屏透明遮罩，实现鼠标拖拽框选区域。
3. **本地 OCR (含数学公式)：**
   - **实现：** 集成 `Pix2Text`。截取图片后，在内存中直接传递给模型，获取 LaTeX 格式的文本。
   - *难点攻克：* `Pix2Text` 模型较大，需研究如何打包 (PyInstaller) 或首次运行自动下载权重。
4. **数据上报：**
   - **实现：** 使用 `requests` 库，将 OCR 结果封装为 JSON，POST 发送给 Member B 的 Go 接口。

#### 开发路径

- **Step 1:** 写一个 Python 脚本，能监听键盘并打印 "Pressed"。
- **Step 2:** 实现截图功能，能把屏幕选区保存为图片。
- **Step 3:** 接入 Pix2Text，把截图变成 LaTeX 字符串。
- **Step 4:** 实现 HTTP POST，把字符串发给 `localhost:8080`。

### 👨‍💻 成员 B：服务端 (Go Backend)

**核心任务：** 构建高性能 API 网关与 RAG 检索引擎。

#### 具体功能与实现方案

1. **RESTful API 设计：**
   - **实现：** 使用 `Gin` 框架。设计 `/api/v1/notes` (增删改查) 和 `/api/v1/chat` (对话) 接口。
2. **数据持久化 (SQL)：**
   - **实现：** 使用 `Gorm` + `SQLite`。设计表结构：`Note` (ID, Content, CreatedAt, Tags)。
3. **向量化与检索 (RAG)：**
   - **实现：**
     - **Embed:** 调用 Ollama 的 Embedding 接口（或本地 SentenceTransformer）将文本转为向量。
     - **Store:** 使用 `chroma-go` 客户端或纯内存向量库，存储向量索引。
     - **Retrieve:** 当用户提问时，计算余弦相似度，召回 Top-K 笔记。
4. **LLM 调度 (Agent)：**
   - **实现：** 接收到 Python 发来的原始 OCR 文本后，构造 Prompt 调用 Ollama (如 Qwen2.5)，要求其生成“标题”、“标签”和“摘要”。

#### 开发路径

- **Step 1:** 搭建 Gin 框架，跑通 Hello World 接口。
- **Step 2:** 定义 SQLite 表结构，实现接收 JSON 并存库。
- **Step 3:** 对接 Ollama API，实现“输入文本 -> 返回 Summary”。
- **Step 4:** 实现向量存取逻辑，完成 RAG 核心闭环。

### 👨‍💻 成员 C：Web 前端 (Vue Dashboard)

**核心任务：** 让枯燥的文本和公式变得美观、可交互。

#### 具体功能与实现方案

1. **知识流 (Waterfall)：**
   - **实现：** 使用 Vue 3 + Tailwind CSS 实现瀑布流布局，展示笔记卡片。
2. **富文本渲染 (Markdown + LaTeX)：**
   - **实现：** 核心难点。使用 `markdown-it` 解析 Markdown，插件集成 `markdown-it-katex` 或 `MathJax`，确保 LaTeX 公式完美渲染。
3. **AI 对话窗口：**
   - **实现：** 仿 ChatGPT 界面。左侧气泡（AI），右侧气泡（用户）。使用流式响应（Fetch API / EventSource）处理 Go 端返回的打字机效果。
4. **数据可视化：**
   - **实现：** 使用 `ECharts`，读取标签数据，生成词云或简单的知识关联图。

#### 开发路径

- **Step 1:** 用 Vite 初始化 Vue 3 项目，配置 Tailwind。
- **Step 2:** 写死一些 Mock 数据（包含复杂公式），调通 LaTeX 渲染。
- **Step 3:** 对接 Go 的 GET 接口，展示真实数据库列表。
- **Step 4:** 实现聊天界面，对接 Go 的 Chat 接口。

## 3. 协作与协调机制 (Coordination)

为了避免“各写各的，最后合不上”，必须遵守以下规则：

### 3.1 接口契约先行 (API First)

在开始写代码前，**成员 A、B、C 必须坐在一起（或在线文档）**，确定 JSON 数据格式。

**示例契约：**

- **上传笔记 (Python -> Go):**

  ```
  POST /api/upload
  {
    "raw_text": "f(x) = x^2",
    "source_image_base64": "...",
    "timestamp": 1715000000
  }
  ```

- **查询笔记 (Vue -> Go):**

  ```
  GET /api/notes
  Response:
  [
    {
      "id": 1,
      "content_md": "# 标题\n\n$$f(x)=x^2$$",
      "tags": ["数学", "函数"]
    }
  ]
  ```

### 3.2 联调节点 (Milestones)

- **节点一：Hello World (第 3 天)**
  - Python 发送字符串 "Hello"，Go 收到并打印，Vue 显示 "Hello"。
  - *目的：打通网络链路。*
- **节点二：OCR 通路 (第 7 天)**
  - Python 截图公式，Vue 上能看到渲染好的公式。
  - *目的：验证核心技术难点（Pix2Text + KaTeX）。*
- **节点三：智能闭环 (第 14 天)**
  - Go 接入 Ollama，实现自动打标签；Vue 实现问答对话。
  - *目的：完成 MVP (最小可行性产品)。*

### 3.3 代码规范与环境

- **Git:** 统一仓库。`/agent` 放 Python 代码，`/backend` 放 Go 代码，`/frontend` 放 Vue 代码。
- **端口约定:** Go 服务固定运行在 `:8080`，Vue 开发环境 `:5173`。
- **模型约定:** 统一安装 Ollama，统一拉取 `qwen2.5:7b` 模型，确保 Prompt 效果一致。

## 4. 风险预案

- **Python 打包体积过大？** -> 成员 A 优先研究精简版依赖，或在开发阶段仅运行源码。
- **OCR 速度慢？** -> 成员 B 增加队列机制，Python 上报后立即返回“处理中”，Vue 前端显示“识别中...”占位符。
- **跨域问题 (CORS)？** -> 成员 B 在 Gin 中全局开启 CORS 中间件，允许 `localhost:5173` 访问。