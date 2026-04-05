# AI Learning

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
ai-learning/
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

| 发布日期 | 类型 | 主题 | 标题 |
|----------|------|------|------|
| 2026-01-03 | article | agent | [LLM Agent 漫游指南：从零构建编码 Agent 的实战经验](articles/agent/hitchhikers-guide-to-llm-agent.md) |
| 2025-12-13 | article | mcp | [Skills vs MCP：为什么我把所有 MCP 迁移到了 Skills](articles/mcp/skills-vs-mcp.md) |
| 2025-12-10 | article | fine-tuning | [rLLM SDK：无需改代码即可训练任何 Agent 程序](articles/fine-tuning/rllm-sdk-agent-rl-training.md) |
| 2025-11-26 | article | agent | [长时运行 Agent 的有效编排](articles/agent/long-running-agent-harness.md) |
| 2025-11-24 | article | agent | [Claude 高级工具使用：搜索、编程式调用与示例](articles/agent/advanced-tool-use.md) |
| 2025-10-01 | article | app-dev | [Claude Code 最佳实践：从环境配置到多会话并行](articles/app-dev/claude-code-best-practices.md) |
| 2025-11-04 | article | mcp | [代码执行 + MCP：构建更高效的 Agent](articles/mcp/code-execution-with-mcp.md) |
| 2025-10-20 | article | agent | [Claude Code 沙箱安全：让 Agent 更安全也更自主](articles/agent/claude-code-sandboxing.md) |
| 2025-10-15 | article | agent | [Manus 的上下文工程实践](articles/agent/manus-context-engineering.md) |
| 2025-10-12 | article | agent | [Agent 2.0：从浅层循环到深层 Agent](articles/agent/shallow-to-deep-agents.md) |
| 2025-10-16 | article | agent | [Agent Skills：用文件和文件夹装备 Agent](articles/agent/agent-skills.md) |
| 2025-09-29 | article | agent | [用 Claude Agent SDK 构建 Agent](articles/agent/claude-agent-sdk.md) |
| 2025-09-29 | article | agent | [高效的 AI Agent 上下文工程](articles/agent/context-engineering-for-agents.md) |
| 2025-09-11 | article | agent | [为 Agent 编写高效工具——用 Agent 来优化](articles/agent/writing-tools-for-agents.md) |
| 2025-07-30 | article | agent | [AI 工程中的 Bitter Lesson：添加结构，然后移除它](articles/agent/bitter-lesson-for-ai-engineering.md) |
| 2025-06-23 | article | agent | [Agent 上下文工程策略：Write / Select / Compress / Isolate](articles/agent/context-engineering-strategies.md) |
| 2025-06-13 | article | agent | [Anthropic 多 Agent 研究系统](articles/agent/multi-agent-research-system.md) |
| 2024-12-19 | article | agent | [构建高效 AI Agent：模式图谱与实践指南](articles/agent/building-effective-agents.md) |
| 2025-10-19 | paper | ml-theory | [从 Pipeline 到 Model-native：Agent AI 范式转变综述](articles/ml-theory/pipeline-to-model-native-agents.md) |
| 2025-03-19 | article | ml-theory | [衡量 AI 完成长任务的能力：指数增长趋势与预测](articles/ml-theory/measuring-ai-long-task-ability.md) |
