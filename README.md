# Multiple Agent For Stories V1

English README:https://github.com/zzh3110296407-cmd/Multiple-Agent-For-Stories-V1-English

Multiple Agent For Stories，中文项目名为 **织境**，是一个面向长篇叙事、互动剧本和故事工程的多智能体协同创作工作台。它的核心不是简单续写文本，而是把故事世界、角色状态、章节结构、记忆、事件、决策、质量检查和连续性检查拆成可追踪的结构化对象，再由多个智能体服务协同推进。

这个开源包包含当前可运行的产品主体：

- FastAPI 后端，支持 JSON-first 本地存储和可选 PostgreSQL 主存储。
- Vite/React 产品工作台，包含普通模式和专家模式。
- 普通模式实际使用的完整 `confirmed-ui` 页面与静态资源。
- 模型网关与模型运行观测层，支持 Qwen/DeepSeek 兼容模型调用。
- 故事设定、世界画布、角色主轴、章节规划、场景写作、质量/连续性门、最终输出、插件输出等入口。
- Prompt-first 故事设定、角色可见性、世界规则时机、章节归档与下一章推进等当前运行约束。
- Story Analyzer 故事分析器模块，用于长文本故事分析和框架抽取。
- PostgreSQL schema、迁移与数据库验证材料。

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
- `database/sql-prototype` - PostgreSQL schema、迁移与验证 SQL。

## 环境要求

推荐方式：

- Docker Desktop 与 Docker Compose。
- 一个可用模型 API Key，通常是 `QWEN_API_KEY` 或 `DEEPSEEK_API_KEY`。

可选本地开发环境：

- Python 3.11+
- Node.js 22+
- PostgreSQL 16+，仅在需要 PostgreSQL 主存储时使用。

## Docker 部署

### 1. 启动 Docker

先打开 Docker Desktop，确认状态为 `Engine running`，并使用 Linux containers。然后在 PowerShell 中检查：

```powershell
docker info
```

只有该命令正常显示 Server 信息后再继续。如果出现 `dockerDesktopLinuxEngine`、`daemon is not running` 或 `failed to connect to the docker API`，请先启动或重启 Docker Desktop。

### 2. 克隆并启动

PowerShell：

```powershell
git clone --depth 1 https://github.com/zzh3110296407-cmd/Multiple-Agent-For-Stories-V1.git
cd Multiple-Agent-For-Stories-V1
Copy-Item .env.example .env
docker compose up --build -d
docker compose ps
```

如果使用 `cmd.exe`：

```bat
git clone --depth 1 https://github.com/zzh3110296407-cmd/Multiple-Agent-For-Stories-V1.git
cd Multiple-Agent-For-Stories-V1
copy .env.example .env
docker compose up --build -d
docker compose ps
```

如果使用 Linux 或 macOS：

```bash
git clone --depth 1 https://github.com/zzh3110296407-cmd/Multiple-Agent-For-Stories-V1.git
cd Multiple-Agent-For-Stories-V1
cp .env.example .env
docker compose up --build -d
docker compose ps
```

首次构建需要下载基础镜像和 Python/Node 依赖，通常需要数分钟。完成后，`docker compose ps` 中 `frontend`、`backend` 应显示 `Up`，`postgres` 应显示 `healthy`。

启动后打开：

- 前端入口：<http://localhost:3000>
- 通过前端代理访问后端健康检查：<http://localhost:3000/health>
- 直接访问后端健康检查：<http://localhost:8000/health>

停止服务时执行 `docker compose down`。该命令不会删除 Docker volumes 中的项目数据。

Docker 默认仅将前端和后端绑定到本机 `127.0.0.1`，不会直接暴露给局域网或公网。需要修改端口时，在 `.env` 中设置 `MAS_FRONTEND_PORT` 和 `MAS_BACKEND_PORT`。如需公网部署，请在前方配置带身份认证和 HTTPS 的反向代理，不要直接公开后端 `8000` 端口。

## 配置模型

启动前编辑 `.env`：

```text
QWEN_API_KEY=your_key_here
QWEN_BASE_URL=https://your-openai-compatible-endpoint/v1
QWEN_MODEL_NAME=your-model-name

DEEPSEEK_API_KEY=
```

模型网关采用服务端允许列表：

- `QWEN_BASE_URL` 会自动成为 Qwen 允许访问的模型主机；远程地址必须使用 HTTPS。
- 如需额外的可信模型主机，在 `MULTIPLE_AGENT_STORIES_MODEL_ENDPOINT_ALLOWLIST` 中用逗号分隔填写主机名。
- 默认只允许 `QWEN_API_KEY`、`DASHSCOPE_API_KEY` 和 `DEEPSEEK_API_KEY` 作为对应模型的密钥引用。额外的环境变量名必须显式写入 `MULTIPLE_AGENT_STORIES_MODEL_KEY_ENV_ALLOWLIST`。
- 网页请求不能临时指定未授权主机或读取任意环境变量，模型响应重定向也不会被跟随。

发布模板默认关闭 LangSmith tracing。开发阶段如开启外部 trace，不要公开私有 prompt、raw response 或 API Key。

## 安全边界

- `POST /api/analyze-stories/imports` 默认最多接收 16 MiB，可通过 `MULTIPLE_AGENT_STORIES_MAX_ANALYZE_STORIES_BODY_BYTES` 调低或调整。
- 分析器交接文件只能从 `MULTIPLE_AGENT_STORIES_ANALYZER_OUTPUT_ROOTS` 指定的根目录导入；多个目录使用分号分隔。相对路径会从第一个允许根目录解析。
- Docker 默认允许目录为 `/workspace/app/data/analyzer_outputs` 和 `/workspace/app/Story Analyzer/data/handoff_exports`。
- 本项目当前按本机单用户工作台部署；若用于多人或公网环境，必须另行增加身份认证、权限控制、速率限制和 HTTPS。

## Docker Hub 镜像命名

本仓库的 Docker 镜像名不带语言目录名。当前 Docker Hub 使用同一个仓库，通过 backend/frontend tag 区分服务：

```text
zihangzhong/multiple-agent-for-stories-v1:backend-latest
zihangzhong/multiple-agent-for-stories-v1:frontend-latest
zihangzhong/multiple-agent-for-stories-v1:backend-1.0.0
zihangzhong/multiple-agent-for-stories-v1:frontend-1.0.1
```

本地构建时如果未设置 Docker Hub 镜像变量，`docker compose` 会使用：

```text
multiple-agent-for-stories-backend:latest
multiple-agent-for-stories-frontend:latest
```

如需按 Docker Hub 镜像名重新构建，可以设置：

```powershell
$env:MAS_BACKEND_IMAGE = "zihangzhong/multiple-agent-for-stories-v1:backend-latest"
$env:MAS_FRONTEND_IMAGE = "zihangzhong/multiple-agent-for-stories-v1:frontend-latest"
docker compose build
```

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

切换前需要按文件名顺序应用 `database/sql-prototype/migrations` 中的全部迁移；具体命令和校验方式见该目录的 `README.md`。默认的 `json_primary` 模式不需要执行数据库迁移。

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
cd "<your clone path>"
python -m pip install -r app\backend\requirements.txt
python -m uvicorn app.backend.main:app --host 127.0.0.1 --port 8000
```

前端：

```powershell
cd "<your clone path>\app\frontend"
npm ci
$env:VITE_API_BASE_URL = "http://127.0.0.1:8000"
npm run dev -- --host 127.0.0.1 --port 5173
```

然后打开 <http://127.0.0.1:5173>。

## 常见问题

- 无法连接 Docker API：先打开 Docker Desktop，等待 `Engine running`，再运行 `docker info`。不要在引擎未启动时继续执行部署命令。
- 首次构建耗时较长：使用 `docker compose logs -f` 查看进度；不要关闭 Docker Desktop。
- 后端未连接：检查 `docker compose ps`、`.env` 和 <http://localhost:8000/health>。
- 页面空白或接口返回 404：执行 `git pull` 后重新运行 `docker compose up --build -d`，确保前端使用最新镜像。
- 模型调用失败：检查模型 Key、base URL、模型名和网络访问。
- 端口冲突：停止占用 `3000` 或 `8000` 的进程，或修改 `docker-compose.yml`。
- 重置本地 Docker 数据：只有在确认要删除本地故事项目时，才删除 Docker volumes。
