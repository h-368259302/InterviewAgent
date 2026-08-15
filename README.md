# InterviewAgent · AI 模拟面试官

> ⚠️ **本项目不是原创项目**，而是基于
> [BMN-zyb/AI_InterviewerAgent](https://github.com/BMN-zyb/AI_InterviewerAgent)
> 的 **魔改 / 二次开发版本**，原项目作者为「北木南」。
> 本仓库在原项目基础上进行了个性化修改，版权与核心代码归属原作者。

<div align="center">

<img src="./images/面试过程.png" alt="AI 模拟面试流程" width="720">

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111%2B-teal)](https://fastapi.tiangolo.com)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.2%2B-orange)](https://github.com/langchain-ai/langgraph)
[![LlamaIndex](https://img.shields.io/badge/LlamaIndex-0.10%2B-green)](https://llamaindex.ai)
[![License](https://img.shields.io/badge/License-MIT-lightgrey)](LICENSE)

</div>

## 目录

- [项目简介](#项目简介)
- [来源声明](#来源声明)
- [功能特性](#功能特性)
- [技术栈与依赖](#技术栈与依赖)
- [环境准备](#环境准备)
- [快速开始](#快速开始)
- [运行项目](#运行项目)
- [API 文档](#api-文档)
- [测试](#测试)
- [项目结构](#项目结构)
- [License](#license)

## 项目简介

InterviewAgent 是一个工程级的 AI 模拟面试 Agent 应用。上传简历并输入岗位 JD 后，系统会自动完成：

```text
JD 解析 -> 简历分析 -> RAG 题库检索 -> 智能出题 -> 多轮面试 -> 逐题评分
-> 评估报告 -> 复习计划
```

项目采用多 Agent DAG 协作、RAG 多路召回 + LLM 精排、短期/长期记忆、动态难度调节等设计，并提供 CLI、REST API、WebSocket 与 Web 界面多种使用方式。

## 来源声明

本项目基于 [BMN-zyb/AI_InterviewerAgent](https://github.com/BMN-zyb/AI_InterviewerAgent) 魔改而来，非独立原创项目。

- 原项目仓库：https://github.com/BMN-zyb/AI_InterviewerAgent
- 原项目作者：北木南
- 本仓库用途：个人二次开发 / 魔改学习版本

当前相对原项目的主要改动：

- `agents/question_planner.py`：补充题目数量不足时的处理说明注释。

本仓库会持续基于原项目进行个性化修改，如需原版功能，请直接使用原项目。

## 功能特性

- **多 Agent DAG 协作**：8 个专职 Agent（意图路由、JD 解析、简历分析、出题、面试、评分、复习计划、闲聊），由 LangGraph 编排。
- **RAG 多路召回 + LLM 精排**：向量检索 + BM25 双路召回，RRF 融合后再由 LLM 重排。
- **Agent 记忆系统**：Redis 短期记忆（会话上下文）+ MySQL 长期记忆（用户画像、薄弱点、历史面试记录）。
- **动态难度调节**：Easy / Medium / Hard 三级难度状态机，根据答题表现动态调整。
- **Skill 技能系统**：可插拔的有状态交互模块（测验、讲解、项目亮点、技术对比）。
- **MCP 协议集成**：通过 MCP 标准协议接入 GitHub、Web 抓取等外部工具。
- **多模态交互**：Web 界面 + 摄像头 + STT 语音识别 + TTS 语音合成。

## 技术栈与依赖

### Python 依赖

项目使用 Python 3.10+，依赖文件：

- [requirements.txt](./requirements.txt)：生产环境依赖
- [requirements-dev.txt](./requirements-dev.txt)：开发 / 测试依赖

主要 Python 包：

| 分类 | 依赖 |
|------|------|
| Agent 框架 | langchain、langchain-core、langchain-community、langgraph、langchain-openai、langchain-dashscope |
| RAG | llama-index、llama-index-core、llama-index-embeddings-dashscope、llama-index-llms-dashscope、llama-index-vector-stores-weaviate、llama-index-retrievers-bm25、llama-index-postprocessor-rankgpt-rerank |
| RAG 评估 | ragas、datasets |
| 向量库 / 检索 | weaviate-client、rank-bm25 |
| LLM / Embedding | dashscope、openai、tiktoken |
| Web | fastapi、uvicorn[standard]、pydantic、pydantic-settings、python-multipart、sse-starlette |
| 存储 | redis、SQLAlchemy、PyMySQL、alembic、cryptography |
| 文件处理 | pypdf、python-docx、beautifulsoup4、requests、httpx |
| 音视频 | faster-whisper、numpy、scipy |
| MCP | mcp、anyio |
| CLI / 工具 | typer[all]、rich、python-dotenv、loguru、tenacity、PyYAML、jinja2 |

开发依赖：pytest、pytest-asyncio、pytest-cov、httpx、black、isort、ruff、mypy、ipython、jupyter。

### 外部服务

| 服务 | 版本要求 | 用途 |
|------|----------|------|
| MySQL | 8.0+ | 长期记忆：用户画像、面试记录 |
| Redis | 6.0+ | 短期记忆：会话上下文 |
| Weaviate | 最新版 | 向量数据库：RAG 向量索引 |
| DashScope API Key | - | 通义千问 LLM + Embedding |
| FFmpeg | 可选 | STT / TTS 音视频格式转换 |
| GitHub Token | 可选 | MCP GitHub 工具 |

## 环境准备

### 1. 安装 Python 依赖

```bash
python -m venv venv

# Linux / macOS
source venv/bin/activate

# Windows
venv\Scripts\activate

pip install -r requirements.txt
```

开发环境额外安装：

```bash
pip install -r requirements-dev.txt
```

### 2. 启动外部服务

```bash
# MySQL
systemctl start mysql

# Redis
systemctl start redis
redis-cli ping   # 应输出 PONG

# Weaviate（本地二进制方式，也可使用 Weaviate Cloud）
./weaviate --host 0.0.0.0 --port 8080 --scheme http
```

首次使用 MySQL 时创建数据库：

```sql
CREATE DATABASE interview_agent CHARACTER SET utf8mb4;
CREATE USER 'interview_agent'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON interview_agent.* TO 'interview_agent'@'localhost';
FLUSH PRIVILEGES;
```

### 3. 配置环境变量

```bash
cp .env.example .env
```

编辑 `.env`，至少需要填写：

```env
DASHSCOPE_API_KEY=sk-xxxxxxxxxxxxxxxxxxxx
MYSQL_HOST=127.0.0.1
MYSQL_PORT=3306
MYSQL_USER=interview_agent
MYSQL_PASSWORD=your_password_here
MYSQL_DATABASE=interview_agent

REDIS_HOST=127.0.0.1
REDIS_PORT=6379

WEAVIATE_URL=http://localhost:8080
```

可选配置：`GITHUB_TOKEN`（MCP GitHub 工具）、`LLM_MODEL`（默认 `qwen-max`）、`EMBEDDING_MODEL`（默认 `text-embedding-v3`）、`APP_HOST`、`APP_PORT`、`PUBLIC_URL`。

### 4. 环境自检

```bash
python scripts/check_env.py
```

### 5. 初始化数据库与 RAG 索引

```bash
# 初始化 MySQL 数据表
python scripts/init_db.py

# 构建 RAG 索引（知识库文件默认位于 rag/knowledge_base/）
python scripts/build_index.py
```

## 运行项目

### CLI 模式

```bash
python -m cli.main interview \
  --jd "岗位 JD 文本或 .txt/.md 文件路径" \
  --resume ./resume.pdf \
  --total 10
```

CLI 参数：

| 参数 | 必填 | 说明 |
|------|------|------|
| `--jd` | 是 | 岗位 JD 文本，或 `.txt` / `.md` 文件路径 |
| `--resume` | 否 | 简历文本，或 `.pdf` / `.txt` / `.md` 文件路径 |
| `--total` | 否 | 出题数量，默认 5 |

### Web 模式

```bash
python -m cli.main serve --host 0.0.0.0 --port 8000
```

或直接使用 uvicorn：

```bash
uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
```

浏览器访问：

```text
http://localhost:8000          # Web 界面
http://localhost:8000/docs     # Swagger API 文档
```

### 一键启动

Linux / macOS 可使用脚本自动检查服务并启动：

```bash
chmod +x scripts/start_services.sh
./scripts/start_services.sh
```

## API 文档

启动 Web 模式后访问 `http://localhost:8000/docs` 查看 Swagger 文档。

主要接口：

| 方法 | 路径 | 说明 |
|------|------|------|
| `POST` | `/api/interview/start` | 启动一场面试 |
| `POST` | `/api/interview/chat` | 发送消息，获取面试官回复 |
| `GET` | `/api/interview/report/{session_id}` | 获取面试评估报告 |
| `WS` | `/api/ws/chat` | WebSocket 实时双向通信 |
| `GET` | `/api/health` | 健康检查 |

## 测试

```bash
pip install -r requirements-dev.txt
pytest
```

## 项目结构

```text
.
├── agents/                  # Agent 层（面试官、出题、评分等）
├── api/                     # FastAPI 服务与路由
├── audio/                   # STT / TTS 语音模块
├── cli/                     # Typer CLI 入口
├── config/                  # 全局配置（settings、logging）
├── frontend/                # Web 前端
├── mcp/                     # MCP 客户端与工具
├── memory/                  # Redis 短期记忆 + MySQL 长期记忆
├── orchestration/           # LangGraph DAG 编排与难度状态机
├── rag/                     # RAG 索引、检索、精排与评估
├── scripts/                 # 初始化、索引构建、环境检查等脚本
├── skills/                  # Skill 技能系统
├── tests/                   # 测试用例
├── .env.example             # 环境变量模板
├── requirements.txt         # 生产依赖
├── requirements-dev.txt     # 开发依赖
└── pyproject.toml           # 项目配置
```

## License

本项目基于 [BMN-zyb/AI_InterviewerAgent](https://github.com/BMN-zyb/AI_InterviewerAgent) 修改而来，版权归原作者所有。原项目 README 标注为 MIT License，本仓库未附带 LICENSE 文件，使用前请以原项目授权为准。
