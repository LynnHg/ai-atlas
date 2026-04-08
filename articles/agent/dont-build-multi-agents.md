# 不要构建多 Agent 系统

> 原文链接：[Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents)
>
> 日期：2025-06-12
>
> 作者：Walden Yan（Cognition / Devin）
>
> 主题：agent, multi-agent, context-engineering
>
> 类型：article

## 核心主题：为什么不应该构建多 Agent 系统

Cognition（Devin 开发团队）创始成员 Walden Yan 从构建长时运行 Agent 的实践出发，提出了两个上下文工程核心原则，并以此论证为什么多 Agent 架构在当前是错误方向。

## 一、上下文工程的两大原则

| 原则 | 内容 | 含义 |
|------|------|------|
| **原则 1** | 共享上下文，共享完整的 Agent trace，而非单条消息 | 子 Agent 必须看到完整的决策历史，而非只是被分配的子任务描述 |
| **原则 2** | 行动携带隐式决策，冲突的决策导致糟糕的结果 | 并行的 Agent 各自做出的隐式选择（如视觉风格）会互相矛盾 |

作者认为这两个原则如此关键，以至于**默认排除任何不遵守它们的 Agent 架构**。

## 二、多 Agent 架构的致命缺陷

以"克隆 Flappy Bird"为例：

- **问题 1（缺少上下文）**：子 Agent 1 误解任务，做出了 Super Mario 风格的背景；子 Agent 2 做的鸟不像游戏素材
- **问题 2（隐式决策冲突）**：即使共享了原始任务，两个子 Agent 无法看到彼此的工作，导致视觉风格不一致
- **根本原因**：决策分散 + 上下文无法在 Agent 之间充分共享

在真实生产系统中，对话通常是多轮的，Agent 需要进行工具调用来决定如何分解任务，任何细节都可能影响对任务的理解——这使得上下文丢失的问题更加严重。

## 三、推荐的架构方案

| 方案 | 描述 | 适用场景 |
|------|------|----------|
| **单线程线性 Agent** | 一个 Agent 串行完成所有子任务，上下文连续 | 大多数场景，能走得很远 |
| **带上下文压缩的线性 Agent** | 引入一个 LLM 专门将历史动作和对话压缩为关键细节、事件和决策 | 超长任务，上下文窗口不够时 |

对于上下文压缩，Cognition 甚至**微调了专用的小模型**来做这件事。这需要投入精力去搞清楚哪些信息是关键的，并构建一个擅长此事的系统。

## 四、现实案例分析

### Claude Code 的子 Agent 设计

- 子 Agent **从不并行运行**，通常只用于回答问题，不写代码
- 原因：子 Agent 缺乏主 Agent 的完整上下文，无法完成定义明确的问题之外的任何事情
- 好处：调查性工作不占主 Agent 的上下文窗口，允许更长的 trace
- 评价：Claude Code 的设计者采取了**刻意简单**的方法

### Edit Apply 模型的教训

- **2024 年做法**：大模型输出 markdown 编辑说明 → 小模型重写文件
- **问题**：小模型经常因微小歧义误解大模型的指令，导致错误编辑
- **现在**：编辑决策和应用**合并为单一模型的单次动作**
- **启示**：将决策和执行分离（即使是在两个模型间）也会引入上下文丢失

### 多 Agent 协作的现状

- 人类通过对话解决分歧效率很高，但这需要**非平凡的智能**
- **2025 年**，多 Agent 协作仍然只能产出脆弱系统
- 决策过于分散，上下文无法在 Agent 之间充分共享
- 目前**没有人在认真解决跨 Agent 上下文传递问题**
- 作者认为这个问题会在单线程 Agent 与人类沟通能力提升后**自然解决**，届时将释放更大的并行性和效率

## 五、核心启示

1. **上下文 > 并行**：宁可串行保持上下文一致，也不要为了并行牺牲可靠性
2. **OpenAI Swarm 和 Microsoft AutoGen 被点名批评**：作者认为它们推广的多 Agent 概念是错误方向
3. **上下文压缩是关键技术**：对于长时运行任务，投资于上下文压缩（甚至微调专用模型）比引入多 Agent 更有效
4. **单 Agent 架构能走很远**：大多数生产场景下，单线程线性 Agent 已经足够
5. **当前模型足够聪明，瓶颈在上下文工程**：2025 年最重要的工程能力是动态管理上下文
6. **决策和执行不要分离**：Edit Apply 模型的教训表明，跨模型传递意图会引入歧义

## 参考资源

- [Building Effective Agents - Anthropic](https://www.anthropic.com/engineering/building-effective-agents)
- [A Practical Guide to Building Agents - OpenAI](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf)
- [Communicative Agents for Software Development](https://arxiv.org/abs/2304.03442)
- [MetaGPT](https://github.com/FoundationAgents/MetaGPT)
- [OpenAI Swarm](https://github.com/openai/swarm)
- [Microsoft AutoGen](https://github.com/microsoft/autogen)
