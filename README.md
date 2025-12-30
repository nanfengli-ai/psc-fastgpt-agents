# PSC FastGPT Agents

本仓库提供两个基于 **FastGPT** 平台的 PSC（Port State Control）智能体工程化落地脚手架：

- **Agent 1：PSC 知识引擎（PSC-KB Agent）**
  - 定位：文档 → QA 的知识引擎，建设与维护 PSC 专题知识库（RAG）。
- **Agent 2：PSC 安检执行与报告助手（PSC-Inspector Agent）**
  - 定位：面向单船的 PSC 安检执行、记录与报告生成助手。

> 设计目标：把「FastGPT 工作台（编排/知识库）」与「你们自有系统（文档库、船舶信息、报告模板等）」通过一层轻量网关服务打通，便于上线、审计与维护。



## 依赖与前置条件

- 已部署（或由天津海事局统一提供）**FastGPT 平台**。
- 你需要两类 Key：  
  - **应用特定 Key**：调用应用对话接口（KB Agent / Inspector Agent 各一个）。  
  - **通用 Key**：可选，用于知识库 CRUD（脚本创建知识库/集合/导入数据）。  
  FastGPT OpenAPI 的鉴权、BaseURL 与 Key 类型说明见官方文档。  
  - OpenAPI 介绍：https://doc.fastgpt.io/docs/introduction/development/openapi/intro  
  - 对话接口（兼容 GPT chat/completions）：https://doc.fastgpt.io/docs/introduction/development/openapi/chat


## 在 FastGPT 中落地两个 Agent


###  创建两个 FastGPT 应用（工作流/Agent）

建议在 FastGPT 工作台分别创建两个应用：
1. **PSC-KB Agent**：以“知识库搜索 + AI 对话”为主，回答法规、缺陷判定、适用条款等。
2. **PSC-Inspector Agent**：以“信息采集 → 检查步骤 → 证据/缺陷记录 → 报告输出”为主。

