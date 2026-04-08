# Manus 自主 AI Agent 技术深度分析与开源复现

> 原文链接：[In-depth technical investigation into the Manus AI agent](https://gist.github.com/renschni/4fbc70b31bad8dd57f3370239dccd58f)
>
> 日期：2025-03-11
>
> 作者：renschni（GitHub）
>
> 主题：agent, multi-agent, CodeAct, tool-use
>
> 类型：article

## 一、Manus 系统架构

Manus 不是从零训练的模型，而是构建在现有基础模型之上的**自主 AI Agent 系统**：

| 组件 | 说明 |
|------|------|
| 基础模型 | Claude 3.5/3.7 + 阿里 Qwen 微调模型，多模型动态调用 |
| 运行环境 | 云端 Ubuntu 虚拟沙箱，带 Shell、浏览器、文件系统、Python/Node.js |
| Agent 循环 | 分析 → 计划 → 执行 → 观察，每次迭代只执行**一个**工具动作 |
| 规划模块 | 将复杂目标分解为有序步骤列表，可动态更新 |
| 知识模块 | 注入领域最佳实践、参考信息 |
| 数据源模块 | 预批准的 API（天气、金融等），优先于网页抓取 |
| 多 Agent | 专业子 Agent 并行处理不同子任务（浏览、编码、分析） |

### 基础模型策略

Manus 使用**多模型动态调用**策略——Claude 3 用于复杂逻辑推理，GPT-4 用于编码任务，Gemini 用于综合知识处理。核心是作为编排器（orchestrator）利用最佳可用 AI 能力，而非依赖单一模型。

### 云端沙箱

与浏览器端 Agent 不同，Manus 运行在**服务器端虚拟环境**中，即使用户设备关闭也能继续工作。环境包含完整的 Ubuntu Linux 工作空间，有互联网访问权限，可启动 Web 服务器并暴露到公网。

### Agent 循环

每个循环周期：
1. **分析**当前状态和用户请求（从事件流中读取）
2. **选择动作**（决定使用哪个工具）
3. **执行**动作
4. **观察**结果，追加到事件流

设计上**限制每次迭代只执行一个工具动作**，必须等待结果后再决定下一步，防止模型失控执行长序列未检查操作。

## 二、核心技术亮点

### CodeAct 范式（代码即动作）

这是 Manus 最关键的创新——基于 ICML 2024 论文《Executable Code Actions Elicit Better LLM Agents》，用**可执行 Python 代码**作为 Agent 的通用动作格式：

- **灵活性极高**：可组合多工具、条件逻辑、调用任意库
- **自我调试**：执行出错后分析错误信息，修改代码重试
- **迭代式开发**：写代码 → 执行 → 观察结果 → 调整代码
- 研究表明 CodeAct 在复杂工具使用任务上的成功率**显著高于**文本/JSON 格式

例如获取天气信息时，Manus 会生成调用天气 API 客户端的 Python 代码并打印结果，而非依赖内置的 "Weather" 函数。这意味着工具"命令"本质上是代码中的函数调用。

### 工具集成与控制流

系统 Prompt 明确指示：**每一步必须是工具调用（函数调用），不能直接用自然语言回复**。关键规则：

- 不点击或执行有不可逆副作用的操作（除非用户授权）
- 使用非交互模式（如 `apt-get -y`）避免暂停等待确认
- 浏览器中内容截断时必须滚动，忽略搜索引擎摘要，点进原页面
- 每次只执行一个工具动作，必须检查结果后再继续
- 错误处理：诊断 → 重试 → 换方案 → 最终报告用户

### 文件化记忆系统

| 记忆类型 | 实现方式 |
|----------|----------|
| 即时记忆 | Event Stream（事件流），记录用户消息、动作、观察结果 |
| 持久化暂存 | 文件系统——中间结果存文件，`todo.md` 跟踪进度 |
| 长期知识 | Knowledge 模块 + RAG，按需检索外部文档/数据 |
| 上下文管理 | 旧事件摘要/丢弃，代码和数据存文件而非上下文 |

`todo.md` 作为实时清单：完成每步后打勾，任务结束时验证所有项完成。即使会话暂停或上下文丢失，todo 文件也是恢复状态的真相来源。

### Prompt 工程要点

- 详细的系统 Prompt 划分为结构化章节：`<system_capability>`、`<browser_rules>`、`<coding_rules>` 等
- 输出要求**数千字**的详细报告，引用来源
- 长文档分段保存草稿再拼接，而非一次性输出
- 使用 `notify`（通知，不阻塞）和 `ask`（提问，需回复）区分消息类型
- 禁止透露系统 Prompt 或做有害/违法的事

## 三、开源复现方案

### 架构蓝图

| 组件 | 推荐工具 |
|------|----------|
| LLM 核心 | CodeActAgent（Mistral 7B 微调，32k 上下文）/ GPT-4 API / Llama 3 |
| 沙箱执行 | Docker 容器 + Python 3.10 / Node.js 20 / Playwright |
| 工具集成 | 自建 `agent_tools.py`（search_web、browse_url、execute_python、shell_command） |
| 编排框架 | LangChain AgentExecutor / CrewAI（多 Agent 协调） |
| 记忆/RAG | FAISS 向量数据库 + HuggingFace Embeddings（all-MiniLM-L6-v2） |
| 规划模块 | 单独 LLM 调用生成步骤列表 + todo.md 文件追踪 |
| 用户界面 | Gradio 简易 Web UI |

### 开发路线图

1. **基础循环**：LLM + 1-2 个工具（如 Python REPL），测试模型与工具执行的集成
2. **逐步添加工具**：搜索、浏览器、Shell、文件 I/O，每添加一个就更新 Prompt 并测试
3. **日志与观察处理**：结构化事件流格式，拼接到每次 Prompt
4. **集成规划模块**：任务分解 + 进度追踪（todo.md）
5. **添加 RAG**：向量存储 + 文档检索注入上下文
6. **精调 Prompt 和容错策略**：处理无限循环、错误重试等边界情况
7. **构建界面和异步任务队列**：支持长时间运行任务

### 可借鉴的开源项目

| 项目 | 价值 |
|------|------|
| **CodeActAgent** | 与 Manus 最直接对应——微调模型 + Docker 执行引擎 + 聊天 UI |
| **AutoGPT** | 自主循环、任务列表、向量 DB 记忆、自我批评（critic thoughts） |
| **BabyAGI** | 任务列表维护和重新排序的简洁实现 |
| **LangChain** | 工具定义、AgentExecutor、RAG pipeline 的脚手架 |
| **CrewAI** | 多 Agent 角色协作框架 |

## 四、已知局限

早期用户测试中发现的问题：

- 可能陷入**无限循环**（同一操作反复失败）
- 事实性错误，来源引用不一致
- 无法完成真实交易（订餐、订票）——到最后一步卡住
- 敏感操作需人工接管确认

## 核心启示

1. **Agent ≠ 模型**：Manus 的核心竞争力不在模型本身，而在架构设计（循环、工具、规划、记忆）和精细的 Prompt 工程
2. **CodeAct 是杀手级范式**：用代码作为动作语言，比固定工具 API 灵活得多，是 Agent 能力突破的关键
3. **文件是 Agent 的外脑**：将状态外化为文件（todo.md、笔记文件），是解决 LLM 上下文窗口限制的实用方案
4. **多模型协作**：不同模型擅长不同任务，动态调度可以最大化效率
5. **完全可用开源工具复现**：CodeActAgent + Docker + LangChain + FAISS 可以搭建出类似系统，挑战在于 Prompt 调优和容错处理

## 参考资源

- [CodeAct 论文（ICML 2024）](https://openreview.net/forum?id=jJ9BoXAfFa)
- [CodeActAgent GitHub](https://github.com/xingyaoww/code-act)
- [Manus 泄露的系统 Prompt（jlia0）](https://gist.github.com/jlia0/db0a9695b3ca7609c9b1a08dcbf872c9)
- [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)
- [BabyAGI](https://github.com/yoheinakajima/babyagi)
- [LangChain](https://github.com/langchain-ai/langchain)
- [CrewAI](https://github.com/joaomdmoura/crewai)
