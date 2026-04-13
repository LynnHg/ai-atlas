# AI Atlas

个人 AI 技术学习笔记与实践项目，系统性地整理 AI 领域的知识、文章、论文和代码示例。

---

## 学习方向

| 方向 | 说明 | 目录 |
|------|------|------|
| **Prompt Engineering** | 提示词工程、系统提示词设计、思维链等技巧 | `*/prompt-engineering/` |
| **AI Agent** | 单 Agent / 多 Agent 架构、工具调用、任务编排 | `*/agent/` |
| **RAG** | 检索增强生成、向量数据库、分块策略、混合检索 | `*/rag/` |
| **MCP** | Model Context Protocol、工具集成、Server 开发 | `*/mcp/` |
| **Fine-tuning** | 模型微调、LoRA、数据集准备、评估方法 | `*/fine-tuning/` |
| **AI 应用开发** | 接入 API 构建产品、流式输出、成本控制、生产实践 | `*/app-dev/` |
| **ML 基础理论** | Transformer、注意力机制、训练原理、模型架构 | `*/ml-theory/` |

## 内容导航

```
ai-atlas/
├── articles/          # 技术文章阅读笔记
├── papers/            # 论文阅读笔记
├── courses/           # 课程学习笔记
├── concepts/          # 概念总结 / 知识卡片
├── demos/             # 代码示例和实践项目
└── resources/         # 工具模板、配置、资源链接
```

### [articles/](articles/) — 技术文章阅读笔记

按主题分类的技术博客、工程文章阅读笔记。每篇笔记包含原文链接、核心要点提炼和个人思考。

### [papers/](papers/) — 论文阅读笔记

AI 领域重要论文的阅读笔记，包含论文核心思想、方法论和关键实验结果。

### [courses/](courses/) — 课程学习笔记

系统性课程的学习记录，按课程名称组织（如 Anthropic 官方教程、DeepLearning.AI 等）。

### [concepts/](concepts/) — 概念总结 / 知识卡片

每个文件聚焦一个核心概念，用简洁的语言解释原理、适用场景和关键要点，便于快速查阅。

### [demos/](demos/) — 代码示例和实践项目

动手实践的代码项目，每个 demo 独立一个文件夹，包含代码、说明和运行指南。

### [resources/](resources/) — 工具、模板、资源收集

Prompt 模板、MCP 工具配置、优质学习资源链接等可复用的参考资料。

## 文件元信息规范

所有笔记文件在头部使用统一的元信息格式，便于检索和交叉引用：

```markdown
> 原文链接：[标题](URL)
> 日期：YYYY-MM-DD
> 主题：agent, multi-agent, prompt-engineering
> 类型：article | paper | concept | course-note
```

## 最近更新

> 仅展示最近 20 条，完整索引见 [articles/](articles/) 和 [papers/](papers/)。

| 添加日期 | 类型 | 主题 | 标题 |
|----------|------|------|------|
| 2026-04-13 | article | agent | [扩展 Managed Agents：将大脑与双手解耦](articles/agent/managed-agents-scaling.md) |
| 2026-04-08 | article | agent | [为什么多 Agent 系统需要记忆工程](articles/agent/memory-engineering-for-multi-agent-systems.md) |
| 2026-04-08 | article | agent | [不要构建多 Agent 系统](articles/agent/dont-build-multi-agents.md) |
| 2026-04-08 | article | agent | [Manus 自主 AI Agent 技术深度分析与开源复现](articles/agent/manus-technical-analysis-replication.md) |
| 2026-04-08 | article | agent | [OpenAI 新 Agent 平台：Responses API、Web Search、Computer Use 与 Agents SDK](articles/agent/openai-agents-platform.md) |
| 2026-04-08 | article | agent | [Amazon Bedrock 多 Agent 协作：Supervisor 模式的全托管实现](articles/agent/bedrock-multi-agent-collaboration.md) |
| 2026-04-08 | article | agent | [构建高效的上下文感知多 Agent 生产框架](articles/agent/efficient-context-aware-multi-agent-framework.md) |
| 2026-04-07 | article | ml-theory | [Gemini 长上下文完全指南：从百万 Token 到多模态应用](articles/ml-theory/gemini-long-context-guide.md) |
| 2026-04-07 | article | agent | [AI Agent 的上下文工程：构建 Manus 的经验教训](articles/agent/manus-context-engineering-official.md) |
| 2026-04-07 | article | agent | [深入理解 Agent 的上下文工程](articles/agent/deep-dive-context-engineering-agents.md) |
| 2026-04-07 | article | agent | [上下文焦虑：AI Agent 如何因感知到的上下文窗口而"恐慌"](articles/agent/context-anxiety-ai-agents.md) |
| 2026-04-07 | paper | prompt-engineering | [上下文工程 2.0：上下文工程的上下文](papers/prompt-engineering/context-engineering-2.0.md) |
| 2026-04-07 | article | prompt-engineering | [上下文工程 vs. 提示词工程：从 2D 到 3D 的升维](articles/prompt-engineering/context-engineering-vs-prompt-engineering.md) |
| 2026-04-05 | article | agent | [LLM Agent 漫游指南：从零构建编码 Agent 的实战经验](articles/agent/hitchhikers-guide-to-llm-agent.md) |
| 2026-04-05 | article | mcp | [Skills vs MCP：为什么我把所有 MCP 迁移到了 Skills](articles/mcp/skills-vs-mcp.md) |
| 2026-04-05 | article | fine-tuning | [rLLM SDK：无需改代码即可训练任何 Agent 程序](articles/fine-tuning/rllm-sdk-agent-rl-training.md) |
| 2026-04-05 | article | agent | [长时运行 Agent 的有效编排](articles/agent/long-running-agent-harness.md) |
| 2026-04-05 | paper | ml-theory | [从 Pipeline 到 Model-native：Agent AI 范式转变综述](papers/ml-theory/pipeline-to-model-native-agents.md) |
| 2026-04-05 | article | ml-theory | [衡量 AI 完成长任务的能力：指数增长趋势与预测](articles/ml-theory/measuring-ai-long-task-ability.md) |
| 2026-04-05 | article | agent | [Manus 的上下文工程实践](articles/agent/manus-context-engineering.md) |
| 2026-04-05 | article | agent | [为 Agent 编写高效工具——用 Agent 来优化](articles/agent/writing-tools-for-agents.md) |
