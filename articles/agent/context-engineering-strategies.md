# Agent 上下文工程策略：Write / Select / Compress / Isolate

> 原文链接：[Context Engineering for Agents](https://rlancemartin.github.io/2025/06/23/context_engineering/)
>
> 日期：2025-06-23
>
> 作者：Lance Martin（LangChain）
>
> 主题：agent, context-engineering, memory, rag, multi-agent
>
> 类型：article

---

## 一、核心类比

Andrej Karpathy 的类比：
- **LLM = CPU**，**上下文窗口 = RAM**（工作记忆）
- **上下文工程 = 操作系统管理 RAM 的策略**

三类需要管理的上下文：
- **Instructions**：提示词、记忆、Few-shot 示例、工具描述
- **Knowledge**：事实、知识库
- **Tools**：工具调用的反馈

> "Context engineering 是在每一步为 Agent 的上下文窗口填入恰好正确的信息的精妙艺术和科学。" — Andrej Karpathy

---

## 二、长上下文的四种失败模式

| 失败模式 | 说明 |
|----------|------|
| **Context Poisoning（中毒）** | 幻觉进入上下文后被后续推理当作事实 |
| **Context Distraction（干扰）** | 过量上下文淹没了模型的训练知识 |
| **Context Confusion（混淆）** | 多余上下文影响模型响应 |
| **Context Clash（冲突）** | 上下文中不同部分互相矛盾 |

---

## 三、四大策略框架

### 3.1 Write（写入上下文）

将信息保存到上下文窗口**外部**，供后续使用。

**Scratchpads（草稿本）**：
- 在任务执行期间持久化信息
- 实现：工具调用写入文件，或运行时 State 对象字段
- 案例：Anthropic 多 Agent 研究系统的 Memory 机制

**Memories（长期记忆）**：跨会话记忆，三种类型：

| 类型 | 说明 | 示例 |
|------|------|------|
| **Episodic（情景）** | 过往行为示例 | Few-shot examples |
| **Procedural（程序）** | 行为指令 | CLAUDE.md、Cursor Rules |
| **Semantic（语义）** | 事实和关系 | 用户偏好、项目知识 |

产品化案例：ChatGPT、Cursor、Windsurf 均有自动生成长期记忆的机制。

### 3.2 Select（选择上下文）

将相关信息**拉入**上下文窗口。

**Memory 选择**：
- 简单方式：固定文件始终加载（CLAUDE.md、Rules）
- 复杂方式：Embedding + 知识图谱索引
- 风险：选错记忆可能比没有记忆更糟（Simon Willison 的 ChatGPT 案例）

**Tool 选择**：
- 工具过多时描述重叠导致混乱
- 对工具描述应用 RAG 可提升选择准确率 **3 倍**

**Knowledge（RAG）**：
- 代码 Agent 是最好的大规模 RAG 实践
- Windsurf 经验：索引 ≠ 检索。需要组合 embedding 搜索 + AST 解析 + grep/文件搜索 + 知识图谱 + 重排序

### 3.3 Compress（压缩上下文）

只保留完成任务所需的 Token。

**Context Summarization（摘要）**：
- Claude Code auto-compact：超 95% 上下文时自动摘要
- 可在特定位置添加（Token 密集型工具后、Agent 间交接时）
- 策略：递归摘要、层次化摘要
- Cognition 用微调模型做摘要——凸显复杂性

**Context Trimming（修剪）**：
- 硬编码规则（移除旧消息）
- 训练过的上下文修剪器（如 Provence）

### 3.4 Isolate（隔离上下文）

将上下文分割到不同空间。

**Multi-agent**：
- 每个子 Agent 有独立上下文窗口、专用工具和指令
- Anthropic 数据：多 Agent 显著优于单 Agent，但 Token 消耗高达 **15x**

**Environment Isolation**：
- HuggingFace CodeAgent：代码在沙箱中运行，Token 密集型对象留在环境中
- 只将选定返回值传回 LLM

**State Object**：
- 运行时 State 的 Schema 设计也是隔离手段
- 只有部分字段暴露给 LLM，其他字段隔离存储

---

## 四、四大策略关系

```
Write（写入）      ←→      Select（选择）
  保存到外部                拉入上下文

          ↕                      ↕

Compress（压缩）   ←→      Isolate（隔离）
  精简 Token                分割上下文空间
```

---

## 五、核心启示

1. **上下文工程的 4 大策略**：Write / Select / Compress / Isolate 是完整的分类框架
2. **长上下文有 4 种失败模式**：Poisoning / Distraction / Confusion / Clash
3. **记忆分 3 种类型**：Episodic / Procedural / Semantic
4. **Memory 选择是未解难题**——选错记忆可能比没有记忆更糟
5. **工具描述的 RAG 可提升 3 倍选择准确率**
6. **代码执行是天然的上下文隔离方案**——重型数据留在沙箱中
7. **索引 ≠ 检索**——需要组合多种技术

---

## 参考资源

- [Karpathy 关于 Context Engineering 的推文](https://x.com/karpathy/status/1937902205765607626)
- [Cognition: Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents)
- [Drew Breunig: How Contexts Fail and How to Fix Them](https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html)
- [LangGraph Memory Concepts](https://langchain-ai.github.io/langgraph/concepts/memory/)
- [Anthropic: Claude Think Tool](https://www.anthropic.com/engineering/claude-think-tool)
- [Reflexion 论文](https://arxiv.org/abs/2303.11366)
- [Generative Agents 论文](https://ar5iv.labs.arxiv.org/html/2304.03442)
- [RAG from Scratch](https://github.com/langchain-ai/rag-from-scratch)
