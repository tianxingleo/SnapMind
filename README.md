# SnapMind (智脑笔记) 🧠



> 从截图到知识库，只需一瞬。
>
> A Local-First AI Knowledge Assistant driven by OCR & LLM.



## 📖 项目简介 (Introduction)



**SnapMind** 是一个面向理工科学生和科研人员的桌面端智能化知识管理工具。它旨在解决“视频/文档学习”与“知识整理”之间的断层。

通过简单的快捷键交互，NoteBrain 能够识别截图中的复杂内容（包括手写体、数学公式），利用本地大模型（LLM）自动进行语义理解、打标签、写摘要，并将其结构化存入本地知识库。通过 RAG（检索增强生成）技术，您可以随时与您的过往笔记进行对话。

**灵感来源：** Google NotebookLM, OPPO 小布记忆, Windows Recall, Pix2Text.



## ✨ 核心特性 (Features)



- **📷 智能截屏 (AI-OCR):** 集成 `Pix2Text`，专为**数学公式 (LaTeX)** 和**中文手写体**优化，识别率远超传统 OCR。
- **🤖 自动整理 (Auto-Organize):** 截图即笔记。LLM 自动为笔记生成标题、摘要和分类标签 (`#线性代数` `#微积分`)。
- **🔒 本地优先 (Local-First):** 数据存储在本地 Markdown 和向量库中。支持连接 **Ollama** 本地模型，隐私无忧。
- **🧠 知识问答 (RAG Chat):** 基于向量检索，你可以问：“我上周记的关于拉格朗日中值定理的笔记在哪里？”，系统会给出答案并溯源。
- **📊 可视化面板 (Dashboard):** 基于 Vue 3 的精美 Web 界面，支持 Markdown/LaTeX 实时渲染和知识图谱展示。



## 🏗 技术架构 (Architecture)



本项目采用 **微服务架构**，充分利用各语言优势：

| **模块**         | **角色** | **技术栈**      | **职责**                                                     |
| ---------------- | -------- | --------------- | ------------------------------------------------------------ |
| **Client Agent** | 感知层   | **Python 3**    | 桌面托盘、全局快捷键监听、屏幕截图、Pix2Text OCR 推理。      |
| **Backend API**  | 逻辑层   | **Go (Golang)** | RESTful API、高并发数据处理、SQLite 元数据管理、向量库 (ChromaDB/Milvus) 交互。 |
| **Frontend UI**  | 交互层   | **Vue 3**       | Web 可视化界面、Markdown/KaTeX 渲染、ECharts 图表、对话交互。 |



### 数据流向



```
用户截图` -> `Python (OCR)` -> `JSON payload` -> `Go (存储 & Embedding)` -> `Vue (展示 & 问答)
```



## 🚀 快速开始 (Quick Start)





### 前置要求 (Prerequisites)



- Python 3.10+
- Go 1.21+
- Node.js 18+ & pnpm
- [Ollama](https://ollama.com/) (建议安装并拉取 `llama3` 或 `qwen2.5` 模型)



### 1. 启动 Go 后端 (The Brain)



Bash

```
cd backend
go mod tidy
# 配置 config.yaml 中的数据库路径
go run main.go
# 服务默认运行在 localhost:8080
```



### 2. 启动 Python 客户端 (The Eyes)



*注意：初次运行需要下载 OCR 模型权重，请保持网络通畅。*

Bash

```
cd agent
pip install -r requirements.txt
python main.py
# 程序将最小化到系统托盘，监听 Alt+A (默认)
```



### 3. 启动 Vue 前端 (The Face)



Bash

```
cd frontend
pnpm install
pnpm dev
# 访问 http://localhost:5173
```



## 📝 使用指南 (Usage)



1. **捕捉灵感：** 当你在看网课或读论文时，按下 `Alt + A` (可在配置中修改)。
2. **框选区域：** 鼠标拖拽框选包含公式或笔记的区域。
3. **后台处理：**
   - 系统会自动识别内容。
   - 右下角弹出通知：“笔记已保存，标签：#数学 #矩阵”。
4. **回顾与交互：**
   - 打开 Web 界面，在瀑布流中查看刚刚的笔记。
   - 在对话框输入：“帮我解释一下刚才那个公式的推导过程”，AI 将基于你的截图内容进行解答。



## 🔧 配置说明 (Configuration)



在 `backend/config.yaml` 中配置核心参数：

YAML

```
server:
  port: 8080

llm:
  provider: "ollama" # 或 "openai"
  base_url: "http://localhost:11434"
  model: "qwen2.5:7b"

database:
  sqlite_path: "./data/notebrain.db"
  vector_store: "chroma"
```



## 🗺️ 开发路线图 (Roadmap)



- [x] **v0.1:** 基础架构搭建 (Py截图 -> Go存储 -> Vue展示)。
- [ ] **v0.2:** 集成 Pix2Text 实现高精度公式识别。
- [ ] **v0.3:** 对接 Ollama 实现笔记自动打标与总结。
- [ ] **v0.4:** 引入 RAG (ChromaDB) 实现本地知识库问答。
- [ ] **v1.0:** 知识图谱可视化 (ECharts) 与 笔记导出 (Obsidian 格式)。



## 🤝 贡献 (Contributing)



欢迎提交 Issue 和 Pull Request！

本项目特别适合对 AI 工程化、全栈开发 感兴趣的开发者参与。

1. Fork 本仓库
2. 新建 Feat_xxx 分支
3. 提交代码
4. 新建 Pull Request



## 📄 许可证 (License)



MIT License.