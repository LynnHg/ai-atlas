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

| 日期 | 类型 | 主题 | 标题 |
|------|------|------|------|
| 2025-09-29 | article | agent | [高效的 AI Agent 上下文工程](articles/agent/context-engineering-for-agents.md) |
| 2025-06-13 | article | agent | [Anthropic 多 Agent 研究系统](articles/agent/multi-agent-research-system.md) |
| 2024-12-19 | article | agent | [构建高效 AI Agent：模式图谱与实践指南](articles/agent/building-effective-agents.md) |
