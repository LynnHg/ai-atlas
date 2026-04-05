# AI 工程中的 Bitter Lesson：添加结构，然后移除它

> 原文链接：[Learning the Bitter Lesson](https://rlancemartin.github.io/2025/07/30/bitter_lesson/)
>
> 日期：2025-07-30
>
> 作者：Lance Martin（LangChain）
>
> 主题：agent, bitter-lesson, architecture, engineering-philosophy
>
> 类型：article

---

## 一、Bitter Lesson 的核心

> "70 年 AI 研究的最大教训：利用计算能力的通用方法最终是最有效的，而且优势巨大。" — Rich Sutton

我们给模型施加的"结构"（基于领域知识的归纳偏置）往往限制模型利用不断增长的计算能力。

Hyung Won Chung（OpenAI）的总结：

> "在当前的计算和数据水平下添加必要的结构。之后移除它们，因为这些捷径会成为进一步改进的瓶颈。"

---

## 二、一个 AI 工程的实例：open-deep-research

### 阶段 1：添加结构（2024 年末）

工具调用不可靠 + 上下文窗口小 → 选择 Workflow 而非 Agent。

施加的假设：
- 请求必须分解为报告章节
- 并行研究和写作以提速
- 避免工具调用以提高可靠性

### 阶段 2：遇到瓶颈（2025 年初）

模型进步暴露了结构的限制：
- 工具调用已变好 → 但代码不用，无法利用 MCP 生态
- 并非所有请求都适合分解为章节 → 代码强制这种策略
- 并行写章节导致报告不连贯 → 代码强制并行写作

### 阶段 3：移除结构

**第一次**：转向多 Agent + 工具调用。但仍保留"每个子 Agent 写自己的章节"→ 报告仍不连贯。

> 常见错误：**未能完全移除**之前添加的结构。

**第二次**：写作移到最后一步。灵活规划研究 → 多 Agent 收集上下文 → 一次性写完整报告。

结果：Deep Research Bench 得分 **43.5**（前 10）。

---

## 三、给 AI 工程师的三条教训

### 教训 1：理解你应用中的结构

问自己：设计中**烘焙了哪些关于 LLM 性能的假设**？

> Jared Kaplan（Anthropic 联合创始人）："构建目前还不太 work 的东西也有好处，因为模型会追上来（通常很快）。"

### 教训 2：随着模型改进重新评估结构

不要固守旧假设。模型在快速进步。

### 教训 3：让结构容易移除

- Agent 抽象可能让移除结构变得更难
- 使用框架的**底层构建块**（如 LangGraph 的 nodes/edges），而非高层抽象
- 框架用于通用功能（如检查点），保持配置灵活

---

## 四、核心启示

1. **Bitter Lesson 适用于 AI 工程**——结构在模型进步后变成瓶颈
2. **问"烘焙了哪些假设"**——每个架构决策都隐含对当时模型能力的假设
3. **构建"还不太 work 的东西"也有价值**——模型会追上来
4. **未能完全移除旧结构是常见错误**——升级到 Agent 但保留 Workflow 的假设
5. **让结构容易移除**——用底层构建块，避免紧耦合的高层抽象
6. **模型会变得更好是唯一可预测的趋势**——围绕这一点设计应用最重要
7. **不要怕重构**——Manus 重构 5 次，open-deep-research 经历 3 次架构升级

---

## 参考资源

- [Rich Sutton: The Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html)
- [Hyung Won Chung 演讲](https://youtu.be/orDKvo8h71o?si=fsZesZuP25BU6SqZ)
- [Boris Cherny 谈 Claude Code 与 Bitter Lesson](https://www.youtube.com/watch?v=Lue8K2jqfKk)
- [open-deep-research 项目](https://github.com/langchain-ai/open_deep_research)
- [Cognition: Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents)
- [Harrison: How to Think About Agent Frameworks](https://blog.langchain.com/how-to-think-about-agent-frameworks/)
- [Jared Kaplan 访谈](https://youtu.be/p8Jx4qvDoSo?si=giFZoWqhevhPn_qu)
