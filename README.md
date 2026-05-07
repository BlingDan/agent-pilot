# Agent-Pilot

![Python](https://img.shields.io/badge/python-3.11+-blue.svg)  ![FastAPI](https://img.shields.io/badge/FastAPI-005571.svg?logo=fastapi&logoColor=white)  ![React](https://img.shields.io/badge/React-20232A?logo=react&logoColor=61DAFB)  ![Flutter](https://img.shields.io/badge/Flutter-02569B?logo=flutter&logoColor=white)  ![Lark](https://img.shields.io/badge/Lark-FF6A00?logo=slack&logoColor=white)

## 项目简介

Agent-Pilot 是一个以 **Feishu/Lark IM 为入口** 的多智能体协作助手。
它将 IM 指令转化为可编排任务流，并通过 `planner / doc / presentation / canvas / delivery` 等 Agent 实现从需求理解、内容生成到结果回传的闭环。

项目同时提供两类可操作界面：

- **Web Cockpit**：任务总览与 Artifact 详情查看
- **Mobile & Windows Clients**：用于移动端/桌面端协作入口

该系统的目标是：

- 降低 IM 自动化流程的上手成本
- 减少跨渠道人工切换，提高任务处理效率
- 在统一的任务语义下支撑不同客户端的一致体验

---

## 解决的问题

- IM 场景下重复劳动高、流程切换多
- 任务结果分散在消息、文档、PPT、Canvas 中无法统一管理
- 多终端状态不一致导致协作效率下降
- 缺少可审计的任务生命周期与回溯能力

Agent-Pilot 通过统一任务模型和多表面适配层，将以上问题收敛到一条可追踪的任务链路。

---

## 核心能力

- 智能任务编排：接收 IM 指令并生成可执行任务计划
- 多 Agent 分工：
  - `intent_router_agent`：意图识别与路由
  - `agent_pilot_planner`：任务拆解与执行计划
  - `doc_agent` / `canvas_agent` / `presentation_agent`：内容与展示产物生成
  - `assistant` / `runtime`：执行编排与状态推进
- 统一状态与快照：`shared` 与 `app.shared.state_service` 管理任务生命周期
- 分层消息入出口：IM、Windows、Mobile、Cockpit 各自独立 surface
- 兼容替代交互：dry-run 模式与 fallback 客户端，支持本地化调试

---

## 目录结构

```text
app/
  assistant/            统一的运行与编排入口（runtime / orchestrator）
  agents/               多 Agent 协作与职责划分
  core/                通用配置与应用启动配置
  integrations/        第三方平台接入（Lark/Feishu + artifact 工具层）
  schemas/             请求/响应与状态 schema
  services/            任务构建、交付与聚合服务
  shared/              通用模型、快照、状态服务
  surfaces/            多表面入口（IM / Web cockpit / Mobile / Windows）

clients/
  agent_pilot_cockpit/ 基于 React + Vite 的 Web cockpit
  agent_pilot_flutter/ Flutter 客户端（Windows + Android）

docs/
  PLAN.md
  Record.md

scripts/
  start_agent_pilot_dev.ps1

tests/
  pytest 套件（按模块覆盖 agents / orchestrator / services / surfaces）
```

---

## 运行前提与环境变量

建议先准备：

- Python 3.11+
- Node.js 18+
- Flutter SDK（若启动移动/桌面客户端）
- 可用的 IM Bot 配置（Feishu/Lark）

项目默认使用 `.env` 管理运行参数。示例变量（请按实际环境配置）：

- `FEISHU_APP_ID`
- `FEISHU_APP_SECRET`
- `OPENAI_API_KEY` 或兼容 OpenAI 的模型服务密钥
- `AGENT_PILOT_AI_MODEL`
- `AGENT_PILOT_MAX_RETRIES`

示例：`cp .env .env.local` 后按需修改。

> 注意：请避免提交含敏感信息的 `.env`。

---

## 快速启动

### 1) 后端服务

```bash
# 安装依赖
pip install -r requirements.txt

# 启动 API 服务
uvicorn app.main:app --reload
```

### 2) IM 事件监听（可选）

```bash
python scripts/lark_event_listener.py
```

### 3) Web Cockpit

```bash
cd clients/agent_pilot_cockpit
npm install
npm run dev
```

### 4) Flutter（Windows）

```bash
cd clients/agent_pilot_flutter
flutter pub get
flutter run -d windows
```

### 5) Flutter（Android 调试）

```bash
adb devices
adb reverse tcp:8000 tcp:8000
flutter run -d <device> --dart-define=AGENT_PILOT_API_BASE=http://127.0.0.1:8000
```

### 6) 一键脚本

```powershell
powershell -ExecutionPolicy Bypass -File scripts\start_agent_pilot_dev.ps1 -StartAndroid -StartScrcpy
```

---

## 主要 API（摘录）

### IM Surface

- `POST /api/im/commands`
- `POST /api/im/events`

### Assistant / Assistant Task

- `GET /api/assistant/tasks/{task_id}`
- `POST /api/assistant/tasks/{task_id}/actions/confirm`
- `POST /api/assistant/tasks/{task_id}/actions/revise`
- `POST /api/assistant/tasks/{task_id}/actions/reset`

### Cockpit Surface

- `GET /api/cockpit/tasks`
- `GET /api/cockpit/tasks/{task_id}`
- `GET /api/cockpit/tasks/{task_id}/artifacts/{kind}`
- `WS /api/cockpit/ws/tasks`
- `WS /api/cockpit/ws/tasks/{task_id}`

### Mobile / Windows Surface

- `GET /api/windows/home`
- `GET /api/windows/tasks/{task_id}`
- `GET /api/mobile/home`
- `GET /api/mobile/tasks/{task_id}`

---

## 清理与维护

### 建议清理命令

```bash
# 删除测试遗留临时目录（已在 .gitignore 中覆盖）
Remove-Item -Recurse -Force .pytest_tmp_manual

# 检查未跟踪文件
git status --short
```

### .gitignore 策略

本仓库在 `.gitignore` 中统一忽略：

- Python 编译缓存与虚拟环境
- Node/Flutter 依赖与构建产物
- pytest 临时目录（含新增的 `*.tmp` 临时族）
- 日志、OS 元数据与 IDE 文件

---

## 测试

```bash
pytest
# 或按模块执行
pytest tests/test_agent_pilot_orchestrator.py
```

---

## 开发约定

- 约定职责边界：`agent` 负责业务行为，`service` 负责组合编排，`shared` 仅提供稳定公共能力。
- 接口变更优先更新对应 `schemas` 与 `tests`。
- 新增 surface 时，优先补齐对应测试文件和文档。

---

## 开源与贡献

欢迎提交 Issue / PR 改进以下方向：

- Agent 能力扩展与优化
- Feishu/Lark 平台能力增强
- Cockpit 观测性与交互优化
- 多端同步体验与离线策略


