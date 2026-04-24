# COM6104 Project Notebook / 课程项目：Moodle AI Agent（MCP Demo）


- 主题：基于 Model Context Protocol（MCP）标准的工具化 Agent 示例（使用虚拟 Moodle 数据 + 可选的 Ollama 总结模型）

### 项目概览

该 Notebook 演示了一个“课程助手”式 AI Agent 的核心组成：

- 一个 MCP 风格的工具服务器（FastAPI）
- 一个短期记忆系统（用于上下文管理）
- 两个工具：
  - `moodle_tool`：模拟 Moodle LMS 数据访问（作业/材料/笔记）
  - `summary_tool`：对课程材料进行总结（优先调用本地 Ollama 模型 `qwen3:0.6b`，不可用时降级）
- 一个 Agent 核心：具备“意图分析 → 规划 → 调用工具 → 汇总输出”的闭环
- 一个 Notebook 内嵌的简易对话界面（HTML/JS）用于演示

### 功能点

- **MCP Server（FastAPI）**
  - `GET /health`：健康检查
  - `GET /tools`：列出已注册工具
  - `POST /tools/call`：以 MCP 风格调用工具

- **短期记忆系统**
  - 使用 `deque(maxlen=...)` 保存最近 N 条记录
  - 自动维护 `context_summary`（最近若干条消息/工具调用摘要）
  - 支持简单的记忆搜索与偏好存取

- **工具 1：`moodle_tool`（虚拟数据）**
  - `check_assignment_status(course_id, assignment_id=None)`
  - `check_new_notes(course_id)`
  - `get_course_materials(course_id)`
  - 内置两门课程的模拟数据（示例课程 ID：`1`、`2`）

- **工具 2：`summary_tool`（总结工具）**
  - 默认模型：`qwen3:0.6b`
  - 优先调用 `ollama` Python SDK；不可用则走简单抽取式降级摘要

- **Agent 核心**
  - 根据用户输入关键词（作业/笔记/材料/总结等）构建执行计划
  - 逐步执行计划并收集工具结果
  - 生成结构化的自然语言响应

### 依赖与环境

Notebook 中出现过的安装依赖如下（按实际需要选择）：

- 必需（服务与工具基础）：
  - `fastapi`
  - `uvicorn`
  - `pydantic`
  - `httpx`
  - `python-dotenv`
  - `websockets`

- Notebook/Jupyter 运行相关：
  - `nest_asyncio`

- 可选（本地 AI 总结）：
  - `ollama`（Python 包）
  - 本机 Ollama 服务（默认：`http://localhost:11434`）

#### 环境变量

Notebook 中读取了以下环境变量（虚拟数据模式下仅用于展示/兼容）：

- `MOODLE_URL`
- `MOODLE_TOKEN`
- `MOODLE_USER_ID`
- `OLLAMA_HOST`（默认 `http://localhost:11434`）
- `OLLAMA_MODEL`（默认 `qwen3:0.6b`）

### 快速运行（推荐：直接跑 Notebook）

1. 打开 `COM6104_project.ipynb`
2. 按顺序运行单元格
3. 在“对话界面”单元格中可直接交互演示（Notebook 内嵌 HTML）

建议输入示例：

- `课程 1 的作业`
- `课程 1 的材料`
- `课程 1 的笔记`
- `总结课程 1 的材料`

### MCP Server API 示例

#### 健康检查

- `GET /health`

```json
{
  "status": "healthy",
  "timestamp": "2026-...",
  "tools_registered": 2
}
```

#### 获取工具列表

- `GET /tools`

```json
{
  "tools": [
    {
      "name": "moodle_tool",
      "description": "Access Moodle LMS... (Mock Data)",
      "inputSchema": {
        "type": "object",
        "properties": {"action": {"type": "string"}, "course_id": {"type": "integer"}},
        "required": ["action", "course_id"]
      }
    }
  ]
}
```

#### 调用工具

- `POST /tools/call`

请求体示例（查询课程 1 的作业，直接调用 `check_assignment_status(course_id=1)` 这一类方法签名）：

```json
{
  "id": "call-001",
  "name": "moodle_tool",
  "arguments": {
    "course_id": 1
  }
}

```
---

## English

- Topic: A tool-augmented Agent demo aligned with the Model Context Protocol (MCP), using mock Moodle data and an optional Ollama-based summarizer.

### Overview

The notebook demonstrates a “course assistant” style AI Agent composed of:

- An MCP-style tool server (FastAPI)
- A short-term memory system (context management)
- Two tools:
  - `moodle_tool`: mock Moodle LMS data access (assignments/materials/notes)
  - `summary_tool`: content summarization (prefers local Ollama model `qwen3:0.6b`, falls back if unavailable)
- An Agent core loop: intent analysis → planning → tool calls → response synthesis
- A simple embedded chat UI (HTML/JS) for demonstration in Jupyter

### Features

- **MCP Server (FastAPI)**
  - `GET /health`: health check
  - `GET /tools`: list registered tools
  - `POST /tools/call`: invoke a tool in an MCP-like format

- **Short-Term Memory**
  - Stores the latest N items via `deque(maxlen=...)`
  - Maintains a `context_summary` for recent turns/tool calls
  - Provides basic search and preference storage

- **Tool 1: `moodle_tool` (Mock Data)**
  - `check_assignment_status(course_id, assignment_id=None)`
  - `check_new_notes(course_id)`
  - `get_course_materials(course_id)`
  - Contains mock data for two courses (example IDs: `1`, `2`)

- **Tool 2: `summary_tool` (Summarizer)**
  - Default model: `qwen3:0.6b`
  - Tries the `ollama` Python SDK first; otherwise uses a simple extractive fallback

- **Agent Core**
  - Builds an execution plan based on keywords (assignment/notes/materials/summarize)
  - Executes steps, aggregates tool results
  - Produces a structured natural-language response

### Dependencies & Environment

Packages shown in the notebook (install only what you need):

- Core (server/tools):
  - `fastapi`
  - `uvicorn`
  - `pydantic`
  - `httpx`
  - `python-dotenv`
  - `websockets`

- Jupyter-related:
  - `nest_asyncio`

- Optional (local AI summarization):
  - `ollama` (Python package)
  - Ollama service on your machine (default: `http://localhost:11434`)

#### Environment Variables

The notebook reads the following variables (in mock mode they are mainly for display/compatibility):

- `MOODLE_URL`
- `MOODLE_TOKEN`
- `MOODLE_USER_ID`
- `OLLAMA_HOST` (default: `http://localhost:11434`)
- `OLLAMA_MODEL` (default: `qwen3:0.6b`)

### Quick Start (Recommended: Run the Notebook)

1. Open `COM6104_project.ipynb`
2. Run cells from top to bottom
3. Use the “Chat Interface” cell to interact via the embedded HTML UI

Suggested prompts:

- `course 1 assignments` / `课程 1 的作业`
- `course 1 materials` / `课程 1 的材料`
- `course 1 notes` / `课程 1 的笔记`
- `summarize course 1 materials` / `总结课程 1 的材料`

### MCP Server API Examples

#### Health Check

- `GET /health`

```json
{
  "status": "healthy",
  "timestamp": "2026-...",
  "tools_registered": 2
}
```

#### List Tools

- `GET /tools`

```json
{
  "tools": [
    {
      "name": "moodle_tool",
      "description": "Access Moodle LMS... (Mock Data)",
      "inputSchema": {
        "type": "object",
        "properties": {"action": {"type": "string"}, "course_id": {"type": "integer"}},
        "required": ["action", "course_id"]
      }
    }
  ]
}
```

#### Call a Tool

- `POST /tools/call`

Example request body (calls a method like `check_assignment_status(course_id=1)` directly):

```json
{
  "id": "call-001",
  "name": "moodle_tool",
  "arguments": {
    "course_id": 1
  }
}

