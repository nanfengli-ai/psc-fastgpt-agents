# PSC FastGPT Agents (Tianjin Maritime Bureau)

本仓库提供两个基于 **FastGPT** 平台的 PSC（Port State Control）智能体工程化落地脚手架：

- **Agent 1：PSC 知识引擎（PSC-KB Agent）**
  - 定位：文档 → QA 的知识引擎，建设与维护 PSC 专题知识库（RAG）。
- **Agent 2：PSC 安检执行与报告助手（PSC-Inspector Agent）**
  - 定位：面向单船的 PSC 安检执行、记录与报告生成助手。

> 设计目标：把「FastGPT 工作台（编排/知识库）」与「你们自有系统（文档库、船舶信息、报告模板等）」通过一层轻量网关服务打通，便于上线、审计与维护。

## 目录结构

```text
.
├─ services/
│  ├─ agent-gateway/            # 对外统一 API：转发到 FastGPT 应用对话接口
│  └─ api-file-library/         # 提供 FastGPT “API 文件库”规范接口（对接你们文档库/文件夹）
├─ fastgpt/
│  ├─ flows/                    # 导出的 FastGPT 工作流 JSON（占位/示例）
│  └─ prompts/                  # 提示词/规范/输出格式
├─ scripts/                     # 可选：FastGPT 知识库 OpenAPI 脚本（创建/导入）
├─ docs/                        # 方案说明、检查清单、报告字段定义
├─ docker-compose.yml           # 本地/内网快速启动（仅网关+文件库服务）
└─ .env.example
```

## 依赖与前置条件

- 已部署（或由天津海事局统一提供）**FastGPT 平台**。
- 你需要两类 Key：  
  - **应用特定 Key**：调用应用对话接口（KB Agent / Inspector Agent 各一个）。  
  - **通用 Key**：可选，用于知识库 CRUD（脚本创建知识库/集合/导入数据）。  
  FastGPT OpenAPI 的鉴权、BaseURL 与 Key 类型说明见官方文档。  
  - OpenAPI 介绍：https://doc.fastgpt.io/docs/introduction/development/openapi/intro  
  - 对话接口（兼容 GPT chat/completions）：https://doc.fastgpt.io/docs/introduction/development/openapi/chat

## 快速开始（网关 + API 文件库）

1) 复制环境变量并按实际填写：

```bash
cp .env.example .env
```

2) 启动服务（本地或内网服务器）：

```bash
docker compose up -d --build
```

3) 健康检查：

- Agent Gateway：`GET http://localhost:8080/healthz`
- API File Library：`GET http://localhost:8081/healthz`

## 在 FastGPT 中落地两个 Agent

### A. PSC 专题知识库（API 文件库方式，推荐）

FastGPT 支持“API 文件库”知识库类型：你只要实现 3 个接口，FastGPT 就能拉取文件列表并选择性导入。  
规范与接口示例见官方文档：  
https://doc.fastgpt.io/docs/introduction/guide/knowledge_base/api_dataset

本仓库的 `services/api-file-library` 已实现该规范，你可以：
- 将 `FILE_LIBRARY_ROOT` 指到你们文档目录（或改造为对接 DMS/对象存储/数据库）
- 在 FastGPT 创建知识库时选择 **API 文件库类型**，填入：
  - baseURL：`http(s)://<your-file-library-host>:8081`
  - Authorization：`Bearer <FILE_LIBRARY_TOKEN>`

### B. 创建两个 FastGPT 应用（工作流/Agent）

建议在 FastGPT 工作台分别创建两个应用：
1. **PSC-KB Agent**：以“知识库搜索 + AI 对话”为主，回答法规、缺陷判定、适用条款等。
2. **PSC-Inspector Agent**：以“信息采集 → 检查步骤 → 证据/缺陷记录 → 报告输出”为主。

FastGPT 支持导出工作流配置（用于模板/版本管理）。导出方式参考：  
https://doc.fastgpt.io/docs/use-cases/app-cases/submit_application_template

- 导出后，把 JSON 粘贴到：`fastgpt/flows/psc-kb-agent.workflow.json`、`fastgpt/flows/psc-inspector-agent.workflow.json`
- 提示词与输出规范建议放在：`fastgpt/prompts/`

### C. 应用对外调用（Agent Gateway）

FastGPT 的对话接口兼容 GPT 的 `/v1/chat/completions`，只需改 BaseURL 与 Authorization。  
本仓库 `services/agent-gateway` 已封装成更适合业务调用的接口：

- `POST /api/kb/ask` → 调用 PSC-KB 应用
- `POST /api/inspector/run` → 调用 PSC-Inspector 应用

配置 `.env`：
- `FASTGPT_BASE_URL`：形如 `http(s)://<fastgpt-host>/api`
- `FASTGPT_KB_APP_KEY`、`FASTGPT_INSPECTOR_APP_KEY`：两应用的 **应用特定 Key**

## 可选：在工作流里反调你们系统（HTTP 节点）

Inspector Agent 往往需要拉取船舶信息、历史缺陷、生成报告编号等。  
FastGPT 工作流提供 HTTP 请求节点，可直接调用你们内部 API。  
参考文档：https://doc.fastgpt.io/docs/introduction/guide/dashboard/workflow/http

## License

建议内部项目使用企业/单位允许的协议（MIT/Apache-2.0 等）。本仓库默认 MIT（可自行替换）。
