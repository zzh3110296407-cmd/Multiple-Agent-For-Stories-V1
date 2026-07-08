# Multiple Agent For Stories V1

Multiple Agent For Stories，中文项目名为 **织境**，是一个面向长篇叙事、互动剧本和故事工程的多智能体协同创作工作台。它的核心不是简单续写文本，而是把故事世界、角色状态、章节结构、记忆、事件、决策、质量检查和连续性检查拆成可追踪的结构化对象，再由多个智能体服务协同推进。

这个开源包包含当前可运行的产品主体：

- FastAPI 后端，支持 JSON-first 本地存储和可选 PostgreSQL 主存储。
- Vite/React 产品工作台，包含普通模式和专家模式。
- 模型网关与模型运行观测层，支持 Qwen/DeepSeek 兼容模型调用。
- 故事设定、世界画布、角色主轴、章节规划、场景写作、质量/连续性门、最终输出、插件输出等入口。
- Story Analyzer 故事分析器模块，用于长文本故事分析和框架抽取。
- PostgreSQL schema 原型、存储契约和数据库迁移基础材料。

本目录由开发工作区自动生成。长期维护时请修改源项目，然后重新生成开源包，不建议直接把生成目录里的文件当作唯一修改源。

## 项目简介

本项目把故事创作拆成一个可审计的多智能体流程：

1. 用户创建或进入故事项目。
2. 系统生成并确认故事设定、世界画布、主角团和全局框架。
3. 章节规划生成轻路线、当前章节概要和场景数量。
4. 场景写作先生成结构化场景草稿，再进入正文生成、修订和确认。
5. 记忆、事件、状态变更、决策、质量检查和连续性检查记录故事状态变化。
6. 最终故事包和插件输出只在用户可见确认后进入导出链路。

目标用户包括小说作者、剧本创作者、互动叙事设计者，以及需要研究多智能体叙事工程的开发者。

## 包含内容

- `app/backend` - FastAPI 后端、服务层、模型、存储适配器和 API 路由。
- `app/frontend` - React/Vite 产品 UI 与工作台页面。
- `app/Story Analyzer` - 故事分析器模块，用于长篇故事分析与框架抽取。
- `database/sql-prototype` - PostgreSQL schema 与迁移原型。
- `database/contracts` - 存储/API 契约文档。
- `database/storage_foundation` - 数据库存储基础与迁移支持文件。

## 默认不包含的内容

历史阶段文件夹、Harness 缓存、调试截图、运行时生成的项目数据、虚拟环境、`node_modules`、构建产物和私有环境变量文件都不会进入默认开源包。

较大的 `app/frontend/public/confirmed-ui` 可视化草稿资源默认也会排除，以便 GitHub 仓库和 Docker build context 保持可控。如果需要普通模式 iframe 的完整视觉草稿页面，可在源项目中用 `--include-design-assets` 重新生成。

## 环境要求

推荐方式：

- Docker Desktop 与 Docker Compose。
- 一个可用模型 API Key，通常是 `QWEN_API_KEY` 或 `DEEPSEEK_API_KEY`。

可选本地开发环境：

- Python 3.11+
- Node.js 22+
- PostgreSQL 16+，仅在需要 PostgreSQL 主存储时使用。

## Docker 部署

在中文开源包目录中执行：

```powershell
cd "<your clone path>\Chinese"
Copy-Item .env.example .env
docker compose up --build
```

如果使用 `cmd.exe`：

```bat
cd /d "<your clone path>\Chinese"
copy .env.example .env
docker compose up --build
```

启动后打开：

- 前端入口：<http://localhost:3000>
- 通过前端代理访问后端健康检查：<http://localhost:3000/health>
- 直接访问后端健康检查：<http://localhost:8000/health>

## 配置模型

启动前编辑 `.env`：

```text
QWEN_API_KEY=your_key_here
QWEN_BASE_URL=https://your-openai-compatible-endpoint/v1
QWEN_MODEL_NAME=your-model-name

DEEPSEEK_API_KEY=
```

发布模板默认关闭 LangSmith tracing。开发阶段如开启外部 trace，不要公开私有 prompt、raw response 或 API Key。

## 存储模式

Docker 默认使用 JSON 主存储：

```text
MULTIPLE_AGENT_STORIES_STORAGE_MODE=json_primary
MULTIPLE_AGENT_STORIES_DATA_DIR=/workspace/app/data/local_project
```

PostgreSQL 作为可选服务随 Docker Compose 一起提供。若要切换为 PostgreSQL 主存储，可设置：

```text
MULTIPLE_AGENT_STORIES_STORAGE_MODE=postgres_primary
MULTIPLE_AGENT_STORIES_DATABASE_URL=postgresql://mas:mas_dev_password@postgres:5432/mas
```

运行时数据保存在 Docker volume 中：

- `app_data` 保存故事项目数据。
- `postgres_data` 保存 PostgreSQL 数据。

## 如何进入项目

1. 打开 <http://localhost:3000>。
2. 查看顶部状态区，确认出现 `backend connected`，并检查模型状态是否健康。
3. 如果模型未配置，进入 `模型设置 / Model Settings` 填写模型信息。
4. 使用左侧导航进入完整故事工作流。

工作台包含两种模式：

- `普通 / Ordinary` - 面向正常使用的产品化流程。
- `专家 / Expert` - 面向调试、诊断和底层状态查看的工作台。

## 基本操作流程

1. `创建项目 / Create Project` - 创建或选择故事项目。
2. `故事设定 / Story Setup` - 生成并确认故事初始设定和交接状态。
3. `世界画布 / World Canvas` - 生成、修订、验证并确认世界级事实基础。
4. `角色 / Character Spine` - 创建 A 级主角团和 B/C/D 配角，必要时构建角色上下文。
5. `框架 / Framework` - 构建全局框架包和章节级骨架。
6. `章节计划 / Chapter Planning` - 生成或修订轻路线、当前章节概要和场景数量。
7. `场景写作 / Scene Writing` - 生成场景草稿，查看质量/连续性结果，修订并确认。
8. `记忆与连续性 / Memory & Continuity` - 检查记忆同步、旧剧情补全和连续性问题处理。
9. `最终输出 / Final Output` - 组装已确认的最终故事包。
10. `插件输出 / Plugin Output` - 在最终故事包准备好后运行插件式输出流程。

重要规则：系统会区分草稿、候选、临时确认和正式确认状态。用户确认、Decision 账本、质量门、记忆同步和连续性检查都是产品核心流程的一部分。

## 不使用 Docker 的本地开发

后端：

```powershell
cd "<your clone path>\Chinese"
python -m pip install -r app\backend\requirements.txt
python -m uvicorn app.backend.main:app --host 127.0.0.1 --port 8000
```

前端：

```powershell
cd "<your clone path>\Chinese\app\frontend"
npm ci
$env:VITE_API_BASE_URL = "http://127.0.0.1:8000"
npm run dev -- --host 127.0.0.1 --port 5173
```

然后打开 <http://127.0.0.1:5173>。

## 常见问题

- 后端未连接：检查 `docker compose ps`、`.env` 和 <http://localhost:8000/health>。
- 模型调用失败：检查模型 Key、base URL、模型名和网络访问。
- 端口冲突：停止占用 `3000` 或 `8000` 的进程，或修改 `docker-compose.yml`。
- 普通模式 iframe 页面缺失：如果需要完整视觉草稿页面，请重新生成并包含 design assets。
- 重置本地 Docker 数据：只有在确认要删除本地故事项目时，才删除 Docker volumes。

