# SnapMind 项目结构与协作指南

本文档规定了 SnapMind 项目的文件目录结构、版本控制规范以及 GitHub 团队协作流程。

## 1. 📂 推荐的文件目录结构 (Monorepo)

本项目采用 **Monorepo (单体仓库)** 模式，即将 Python 客户端、Go 服务端和 Vue 前端代码存放在同一个 Git 仓库中。这样便于统一版本管理、联调和文档维护。

```
SnapMind/
├── .git/
├── .gitignore             # 全局忽略文件 (合并 Python, Go, Node 规则)
├── README.md              # 项目总说明
├── LICENSE
│
├── docs/                  # 📚 项目文档
│   ├── api_contract.md    # [核心] API 接口契约文档 (Member A, B, C 共同维护)
│   ├── database_schema.sql # 数据库设计文档
│   └── dev_setup.md       # 环境搭建指南
│
├── agent/                 # 🐍 [Member A] Python 桌面端 (感知层)
│   ├── src/
│   │   ├── main.py        # 入口文件
│   │   ├── core/          # 核心逻辑 (截图、快捷键监听)
│   │   ├── ocr/           # OCR 模块 (Pix2Text 封装)
│   │   └── ui/            # PyQt/Pystray 界面代码
│   ├── assets/            # 图标、提示音等静态资源
│   ├── tests/             # 单元测试
│   ├── requirements.txt   # 依赖清单
│   └── config.py          # 客户端配置
│
├── backend/               # 🐹 [Member B] Go 服务端 (逻辑层)
│   ├── cmd/
│   │   └── server/
│   │       └── main.go    # Go 程序入口
│   ├── internal/          # 私有应用代码
│   │   ├── api/           # Gin 路由与 Handler
│   │   ├── models/        # Gorm 数据库模型
│   │   ├── service/       # 业务逻辑 (LLM调用, RAG检索)
│   │   └── repository/    # 数据库/向量库操作
│   ├── pkg/               # 公共工具库 (可复用代码)
│   ├── config/            # 配置文件 (config.yaml)
│   ├── go.mod             # Go 依赖管理
│   └── go.sum
│
└── frontend/              # ⚡️ [Member C] Vue 前端 (展示层)
    ├── public/            # 静态资源
    ├── src/
    │   ├── api/           # Axios 请求封装 (对接 Go 后端)
    │   ├── components/    # Vue 组件 (Markdown渲染器, 聊天气泡)
    │   ├── views/         # 页面视图 (瀑布流页, 设置页)
    │   ├── App.vue
    │   └── main.js
    ├── index.html
    ├── package.json       # npm 依赖管理
    ├── tailwind.config.js # Tailwind 配置
    └── vite.config.js     # Vite 配置
```

## 2. 🛠 仓库管理规范

### 2.1 全局 `.gitignore` 配置

由于根目录下混合了多种语言，根目录的 `.gitignore` 必须包含所有技术栈的忽略规则：

```
# Python
__pycache__/
*.pyc
agent/venv/
agent/.env

# Go
backend/bin/
backend/dist/
backend/*.exe
backend/*.app

# Node/Vue
frontend/node_modules/
frontend/dist/
frontend/.env.local

# IDE
.vscode/
.idea/
.DS_Store

# Large AI Models (重要！防止上传几百MB的模型文件)
*.onnx
*.pt
*.bin
```

### 2.2 大文件管理 (Git LFS)

如果必须上传 AI 模型权重或大型测试图片，请**务必使用 Git LFS**，不要直接 commit 到 git。

```
git lfs install
git lfs track "*.pt"
git lfs track "*.onnx"
```

## 3. 🤝 GitHub 协同工作流

### 3.1 分支策略 (Branching Strategy)

采用简化的 **GitHub Flow**：

- **`main`**: 主分支。永远保持可运行、可演示的状态（MVP 稳定版）。
- **`dev`**: 开发总干。各成员的代码合并到这里进行联调。
- **Feature Branches**: 成员开发分支。
  - 命名规范：`feat/member-a/screenshot`、`fix/member-b/api-bug`、`feat/member-c/ui-chat`。

### 3.2 协作流程 (Workflow)

**场景：Member B 需要开发一个新的 API 接口**

1. **认领任务 (Issues):** 在 GitHub Issues 中创建一个任务：“[后端] 实现笔记上传接口”，并 Assign 给自己。
2. **创建分支:** `git checkout -b feat/member-b/upload-api`
3. **提交代码:** `git commit -m "feat: add upload handler and sqlite model"`
4. **发起 PR (Pull Request):**
   - 将分支推送到 GitHub。
   - 发起 Pull Request 合并向 `dev` 分支（**注意：不是 main**）。
   - **Code Review:** 在微信群里喊一声“Member A/C，帮我 Review 一下代码”。
   - **Review 重点:** 接口格式是否符合 `/docs/api_contract.md` 中的约定。
5. **合并:** Review 通过后，Squash and Merge 到 `dev` 分支。

### 3.3 联调机制

由于三人分工明确，联调是最大的痛点。

- **Mock 数据先行：**
  - **Member B** 在 API 开发完成前，先提供一份 JSON 样例。
  - **Member C** 在 Vue 代码中硬编码这份 JSON 数据进行 UI 开发。
  - **Member A** 写 Python 脚本时，先打印 JSON，而不是直接发请求。
- **每周同步会：**
  - 每周五晚上开 30 分钟会议，演示各自进度，并合并 `dev` 到 `main`（如果稳定）。

## 4. 📝 初始设置 Checklist (项目启动第一天)

请按照以下顺序初始化仓库：

1. [ ] **创建仓库:** 由一人在 GitHub 创建空仓库 `SnapMind`。
2. [ ] **提交结构:** 将上述目录结构（空文件夹和 `.gitkeep`）提交上去。
3. [ ] **提交文档:** 上传 `README.md` 和 `docs/api_contract.md`（空的也要传，这很重要）。
4. [ ] **配置保护:** 在 GitHub Settings -> Branches 中，将 `main` 分支设为 Protected（禁止直接 Push，必须通过 PR 合并）。
5. [ ] **邀请成员:** 将另外两人添加为 Collaborators。