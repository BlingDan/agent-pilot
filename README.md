<div align="center">
  <img src="./assets/img/agent-pilot.png" alt="Agent-Pilot hero image" width="100%" />
  <h1>Agent-Pilot</h1>
  <h3>基于 IM 的办公协同智能助手</h3>
  <p><em>通过 IM 为入口，完成任务编排、内容生成与结果交付</em></p>
  <p>
    <img src="https://img.shields.io/badge/python-3.11+-blue.svg" alt="Python" />
    <img src="https://img.shields.io/badge/FastAPI-005571?style=flat&logo=fastapi&logoColor=white" alt="FastAPI" />
    <img src="https://img.shields.io/badge/React-20232A?style=flat&logo=react&logoColor=61DAFB" alt="React" />
    <img src="https://img.shields.io/badge/Flutter-02569B?style=flat&logo=flutter&logoColor=white" alt="Flutter" />
    <img src="https://img.shields.io/badge/language-Chinese-brightgreen?style=flat" alt="Language" />
  </p>
</div>

## 项目简介

Agent-Pilot 是一个面向 Feishu/Lark 的 IM-First 自动化助手平台，负责把自然语言任务指令转化为结构化任务流，并驱动多 Agent 协同完成内容生成与投递。

典型能力包括：任务理解、任务拆解、内容产物生成（文档/画布/汇报稿）、任务状态跟踪和结果回写。

## 这份 README 建议写什么

推荐保留以下结构，便于后续长期维护：

- 项目定位：你做了什么、解决什么问题
- 核心能力：系统能做的事
- 目录结构：代码如何组织
- 快速启动：最小可运行步骤
- 环境变量与配置：部署前准备
- API 说明：对接和二次开发入口
- 测试与开发约定：如何保证质量
- 贡献方式：团队协作入口

## 核心能力

- IM 任务接入：通过 IM 事件与命令触发统一任务链路
- 多 Agent 分工协作：意图路由、规划、文档、Canvas、演示稿、交付
- Cockpit 任务管理：任务列表、状态、artifact 详情与回看
- 多终端一体化：Windows / Android 客户端共用任务状态模型
- 可观测与回放：任务状态和快照可追踪

## 目录结构

```text
app/
  assistant/            统一编排与运行时入口（orchestrator / runtime）
  agents/               多 Agent 实现
  core/                 公共配置与应用基础能力
  integrations/         平台与工具接入层（Feishu/Lark 等）
  schemas/              任务 / 请求 / 响应数据模型
  services/             业务服务层（任务构建、交付）
  shared/               公共模型、快照、状态服务
  surfaces/             表面适配层（IM / Cockpit / Mobile / Windows）

clients/
  agent_pilot_cockpit/  React + Vite 前端（任务中控）
  agent_pilot_flutter/  Flutter 客户端（Windows + Android）

scripts/
  start_agent_pilot_dev.ps1  本地联调脚本

docs/
  PLAN.md
  Record.md

tests/
  主要模块与接口测试
```

## 启动与开发

### 1. 安装依赖与启动后端

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload
```

### 2. 启动 IM 事件监听（可选）

```bash
python scripts/lark_event_listener.py
```

### 3. 启动 Cockpit

```bash
cd clients/agent_pilot_cockpit
npm install
npm run dev
```

### 4. 启动 Flutter 客户端

```bash
cd clients/agent_pilot_flutter
flutter pub get
flutter run -d windows
```

Android 调试：

```bash
adb devices
adb reverse tcp:8000 tcp:8000
flutter run -d <device> --dart-define=AGENT_PILOT_API_BASE=http://127.0.0.1:8000
```

### 5. 本地开发脚本

```powershell
powershell -ExecutionPolicy Bypass -File scripts\start_agent_pilot_dev.ps1 -StartAndroid -StartScrcpy
```

## 环境变量（示例）

- `FEISHU_APP_ID`
- `FEISHU_APP_SECRET`
- `OPENAI_API_KEY`（或兼容 OpenAI 的模型服务密钥）
- `AGENT_PILOT_AI_MODEL`
- `AGENT_PILOT_MAX_RETRIES`

> 约定：`.env` 不提交到仓库，敏感配置请放到本地环境中。

## API 快速索引

- `POST /api/im/commands`
- `POST /api/im/events`
- `GET /api/assistant/tasks/{task_id}`
- `POST /api/assistant/tasks/{task_id}/actions/confirm`
- `POST /api/assistant/tasks/{task_id}/actions/revise`
- `POST /api/assistant/tasks/{task_id}/actions/reset`
- `GET /api/cockpit/tasks`
- `GET /api/cockpit/tasks/{task_id}`
- `GET /api/cockpit/tasks/{task_id}/artifacts/{kind}`
- `WS /api/cockpit/ws/tasks`
- `WS /api/cockpit/ws/tasks/{task_id}`
- `GET /api/windows/home`
- `GET /api/windows/tasks/{task_id}`
- `GET /api/mobile/home`
- `GET /api/mobile/tasks/{task_id}`

## 测试

```bash
pytest
```

## 参与贡献

欢迎提交 Issue / PR，改进点包括：

- Agent 能力与任务模型
- Feishu/Lark 平台接入能力
- Cockpit 或客户端交互体验
- 可靠性、异常处理、自动化测试
