# 高效的 AI Agent 上下文工程

> 原文链接：[Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
>
> 日期：2025-09-29
>
> 作者：Prithvi Rajasekaran, Ethan Dixon, Carly Ryan, Jeremy Hadfield
>
> 主题：agent, context-engineering, prompt-engineering, long-horizon-tasks
>
> 类型：article

---

## 一、从 Prompt Engineering 到 Context Engineering

### 1.1 范式转变

- **Prompt Engineering**：关注如何写好系统提示词，适合一次性分类/生成任务
- **Context Engineering**：关注整个上下文状态的策划和管理——系统指令、工具、MCP、外部数据、消息历史等

核心定义：

> Context Engineering = 在有限的上下文窗口中，找到**最小的高信号 Token 集合**，最大化期望结果的概率。

### 1.2 关键区别

| 维度 | Prompt Engineering | Context Engineering |
|------|-------------------|---------------------|
| 关注点 | 如何写好提示词 | 如何策划最优的上下文组合 |
| 时机 | 一次性编写 | 每次推理时动态策划 |
| 范围 | 系统提示词 | 系统指令 + 工具 + MCP + 外部数据 + 消息历史 |
| 适用场景 | 一次性任务 | 多轮长时运行的 Agent |

---

## 二、为什么上下文工程至关重要

### 2.1 Context Rot（上下文腐烂）

随着 Token 数量增加，模型准确回忆信息的能力会下降。原因：

- **注意力预算有限**：每个 Token 需要关注所有其他 Token，产生 n² 的注意力关系
- **上下文越长，注意力越"摊薄"**：性能不是断崖式下降，而是渐进式衰减
- **训练数据偏差**：短序列在训练数据中更常见，模型对长距离依赖的处理能力较弱

### 2.2 核心认知

上下文必须被当作**有限资源**，存在**边际递减效应**——不是越多越好。每增加一个 Token 都在消耗"注意力预算"。

---

## 三、高效上下文的四大组成

### 3.1 系统提示词：找到"正确的高度"

Goldilocks Zone（金发姑娘区间）：

| 极端 | 问题 |
|------|------|
| 过于具体：硬编码复杂 if-else 逻辑 | 脆弱、难维护 |
| 过于笼统：模糊的高层指导 | 模型缺乏具体信号，假设了不存在的共享上下文 |
| **最佳**：具体到能引导行为，又灵活到能提供启发式规则 | 可控且适应性强 |

实践建议：

- 使用 XML 标签或 Markdown 标题分隔不同区域（`<instructions>`、`## Tool guidance`、`## Output description`）
- 追求**完整描述期望行为的最小信息集**（最小 ≠ 最短）
- 先用最好的模型 + 最简 Prompt 测试，根据失败模式再逐步添加

### 3.2 工具设计

- 工具应**自包含、健壮、用途清晰**，像好代码中的函数
- 返回结果要 Token 高效
- 最常见失败模式：**工具集臃肿、功能重叠**

> 判断标准：如果人类工程师都无法确定该用哪个工具，Agent 也做不到。

### 3.3 示例（Few-shot）

- **不要**堆砌大量边缘情况规则
- **要**精选一组多样的、典型的示例展示期望行为
- "对 LLM 来说，示例就是值千字的图片"

### 3.4 总体原则

> **Informative, yet tight**（信息充分，但精炼）

---

## 四、上下文检索：从预计算到 Just-in-Time

### 4.1 传统方式：预检索

预先通过 embedding 检索相关数据塞入上下文。问题：可能过时、冗余、无法适应动态需求。

### 4.2 Just-in-Time 检索

Agent 只保存**轻量标识符**（文件路径、查询语句、链接），在运行时通过工具**按需加载**数据。

Claude Code 的做法：
- 模型写精准查询，存储结果
- 用 `head`、`tail` 等命令分析大文件，而非全部加载到上下文
- 模拟人类认知——我们不记忆整个语料库，而是建立索引系统按需检索

### 4.3 渐进式发现（Progressive Disclosure）

让 Agent 逐层发现上下文，每次交互产生的上下文为下一个决策提供信息：

- 文件大小暗示复杂度
- 命名规范暗示用途
- 时间戳代理相关性

Agent 一层层构建理解，只在工作记忆中保留必要信息。

### 4.4 混合策略（推荐）

最有效的方案是**混合**：

- **预加载**：已知的关键上下文（如 CLAUDE.md）
- **实时获取**：通过 glob、grep 等工具按需检索

> Claude Code 就是混合模型的代表：CLAUDE.md 预加载到上下文，同时用工具实时导航文件系统。

### 4.5 权衡

| 策略 | 优点 | 缺点 |
|------|------|------|
| 预检索 | 速度快 | 可能过时、冗余 |
| Just-in-Time | 新鲜、精准 | 速度慢、依赖工具设计质量 |
| 混合 | 兼顾速度和精准 | 需要判断预加载边界 |

> Anthropic 的建议：**"做最简单的有效方案"（Do the simplest thing that works）**

---

## 五、长时任务的三大策略

### 5.1 Compaction（压缩）

对话接近上下文窗口限制时，总结内容并用摘要重新初始化新的上下文窗口。

Claude Code 的实现：
- **保留**：架构决策、未解决的 Bug、实现细节
- **丢弃**：冗余的工具输出和消息
- 压缩后继续，附带最近访问的 5 个文件

调优建议：
1. 先最大化**召回率**（确保捕获所有相关信息）
2. 再提升**精确度**（消除多余内容）
3. 最安全的轻量压缩：**清除旧的工具调用和结果**

### 5.2 结构化笔记（Agentic Memory）

Agent 定期将笔记写入上下文窗口外的**持久存储**，稍后按需拉回。

典型模式：
- Claude Code 维护 to-do list
- 自定义 Agent 维护 NOTES.md 文件
- 跨上下文重置后读取自己的笔记继续执行

案例——Claude 玩宝可梦：
- 跨数千步追踪目标（"过去 1234 步我在训练皮卡丘，已升 8 级，目标 10 级"）
- 自发建立地图、记录战斗策略、追踪成就
- 上下文重置后读取笔记，无缝继续多小时的策略执行

### 5.3 子 Agent 架构

专门的子 Agent 用干净的上下文窗口处理聚焦任务：

- 每个子 Agent 可能消耗数万 Token 进行深度探索
- 但只返回 **1,000-2,000 Token** 的浓缩摘要
- 主 Agent 专注于综合分析，详细搜索上下文隔离在子 Agent 内

### 5.4 三种策略的适用场景

| 策略 | 最适合 |
|------|--------|
| Compaction | 需要大量来回交互的对话式任务 |
| 结构化笔记 | 有明确里程碑的迭代开发 |
| 子 Agent 架构 | 可并行探索的复杂研究和分析 |

---

## 六、核心启示

1. **上下文是有限资源，有边际递减效应**——不是越多越好，要精心策划
2. **"最小高信号 Token 集合"是核心目标**——每个 Token 都应该有价值
3. **系统提示词要找"正确的高度"**——不要过于具体也不要过于笼统（Goldilocks Zone）
4. **工具集要精简，描述要清晰**——臃肿的工具集是最常见的失败模式
5. **从预检索转向 Just-in-Time 检索**——让 Agent 像人一样按需获取信息
6. **长时任务用三板斧**：Compaction + 结构化笔记 + 子 Agent
7. **"做最简单的有效方案"**——模型越强，需要的工程越少
8. **示例比规则更有效**——精选典型示例 > 堆砌边缘情况规则

---

## 参考资源

- [Anthropic Prompt Engineering Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Building effective AI agents](https://www.anthropic.com/research/building-effective-agents)
- [Writing tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Context Rot 研究](https://research.trychroma.com/context-rot)
- [MCP 协议介绍](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Memory and Context Management Cookbook](https://platform.claude.com/cookbook/tool-use-memory-cookbook)
- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
