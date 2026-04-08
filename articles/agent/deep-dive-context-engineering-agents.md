# 深入理解 Agent 的上下文工程

> 原文链接：[Deep Dive into Context Engineering for Agents](https://galileo.ai/blog/context-engineering-for-agents)
>
> 日期：2025-09-24
>
> 作者：Pratik Bhavsar（Galileo Labs）
>
> 主题：agent, context-engineering, context-window, multi-agent
>
> 类型：article

## 一、从 Prompt Engineering 到 Context Engineering 的范式转变

文章的核心论点是：**管理进入 Agent 上下文窗口的内容，比选择哪个模型更重要**。这一转变的背景：

- Cognition（Devin 团队）明确指出："上下文工程实际上是构建 AI Agent 的工程师的**第一要务**"
- 生产环境的 AI Agent 平均每生成 **1 个 token 就需要处理 100 个输入 token**
- 大多数团队仍然把上下文当"垃圾抽屉"——什么都往里塞，然后祈祷好运

## 二、上下文 = RAM，记忆 = 硬盘

| 概念 | 类比 | 特性 | 说明 |
|------|------|------|------|
| **上下文 (Context)** | 计算机 RAM | 易失性、即时可用 | 当前推理的工作记忆，窗口关闭即丢失 |
| **记忆 (Memory)** | 硬盘 | 持久化、需要显式检索 | 长期存储，跨会话保留，需要主动调取 |

随着上下文窗口扩大和检索技术进步，两者的边界正在模糊，这更加突显了有意设计记忆系统和检索策略的必要性。

## 三、四大上下文失败模式

| 失败模式 | 含义 | 典型症状 |
|----------|------|----------|
| **Context Poisoning（上下文投毒）** | 错误/腐坏的数据进入上下文，污染推理 | Agent 基于错误前提做决策 |
| **Context Distraction（上下文干扰）** | 不相关信息稀释了有效信号 | Agent 注意力被无关内容分散 |
| **Context Confusion（上下文混淆）** | 模糊或重叠的信息让模型困惑 | Agent 无法区分相似但不同的概念 |
| **Context Clash（上下文冲突）** | 来自不同来源的矛盾信息 | Agent 在相互矛盾的指令间摇摆 |

**Context Rot** 研究显示，即使是 GPT-4o，仅因信息呈现方式不同，准确率就能从 **98.1% 暴跌至 64.1%**。性能退化远在 token 上限之前就开始了——recall 变得不可靠、延迟上升、有用信号被过时或冗余的细节淹没。

## 四、五大上下文管理策略

### 策略一：卸载与按需检索 (Offload & Retrieve)

把大块内容移出上下文窗口，只保留轻量指针：

- 网页 → 替换为 URL
- 长文件读取 → 替换为文件路径
- 大量命令输出 → 写入磁盘后引用

**核心优势**：比截断更好——截断立即销毁信息，而卸载保留了随时恢复的能力（可逆压缩）。

在检索策略上，越来越多的 Agent 系统从"预加载一切"的传统 RAG 转向 **Just-in-Time Retrieval（即时检索）**：

- 用 `rg` 搜符号 → 查看文件 → 缩小范围再搜
- 模拟人类在代码库中工作的方式：迭代查询而非预加载
- RAG 并非无用，但不再是唯一默认选择

### 策略二：摘要压缩 (Summarize)

摘要应该是**最后一道防线**，而非第一选择。安全的摘要模式：

1. 将完整对话历史转储到持久存储
2. 用结构化摘要替换上下文中的长历史
3. 尽量保留最近的完整工具交互
4. 允许 Agent 在需要精确细节时重新读取存档

这样"有损摘要"就变成了"可恢复的摘要"。Claude Code 的 `/compact` 命令就是这一设计理念的体现。

### 策略三：子 Agent 隔离 (Context Isolation)

用子 Agent 作为"上下文防火墙"：

- 每个子 Agent 拥有独立的上下文窗口、指令和工具权限
- 冗长的探索性工作不会污染主线程
- 主 Agent 只接收返回的摘要或产物
- 可以对简单的只读任务使用更廉价的模型
- 权限可以收窄以实现更安全的委派

子 Agent 不仅是"工人"，更是一个**防止某部分任务污染另一部分活跃记忆的边界**。

### 策略四：裁剪 (Pruning)

主动移除过时/不相关的上下文内容：

- 基于任务感知的自适应裁剪
- 研究（SWE-Pruner）表明可减少 **23-54%** 的 token
- 保留语法和逻辑结构完整性

### 策略五：缓存 (Caching)

利用 prompt caching 节省成本和延迟，但缓存是脆弱的——只在前缀稳定时才有效。设计原则：

- 保持 **append-only** 的对话历史，不修改旧内容
- 系统提示词保持稳定
- JSON 序列化保持确定性
- 动态元数据与稳定前缀隔离
- **成本节省可达 80-90%**，延迟降低 **85%**

```json
{
  "cache_control": {
    "type": "ephemeral",
    "ttl": "1h"
  }
}
```

一个 15 次调用的 Agent 会话，使用 5,000 token 的系统提示词：不缓存需处理 75,000 token；使用缓存则处理 5,000 次 + 14 次缓存读取，输入成本降低约 **80%**。

## 五、上下文工程决策框架

| 症状 | 推荐策略 |
|------|----------|
| Agent 在多轮对话后遗忘上下文 | 消息裁剪 (Message Trimming) |
| Agent 忽略特定指令 | 结构化系统提示词 (Structured System Prompts) |
| API 成本随轮次线性增长 | Prompt 缓存 (Prompt Caching) |
| Agent 混淆不同任务的发现 | 子 Agent 隔离 (Sub-agent Isolation) |

建议优先级：先从结构化系统提示词开始（零成本），超过 10 轮时加消息裁剪，超过 5 次 API 调用时加缓存，需要多路径探索时加子 Agent 隔离。

## 六、多策略组合

强大的 Agent 系统通常**组合多层策略**，而非依赖单一银弹：

1. **卸载**大块数据
2. **按需检索**细节
3. 仅在必要时**摘要**
4. 用子 Agent **隔离**嘈杂的工作
5. 保持稳定前缀可**缓存**

这比押注更大的上下文窗口更现实。更大的窗口有帮助，但无法解决噪声管理、检索策略、缓存行为和编排卫生问题。

## 核心启示

1. **上下文窗口不是垃圾桶**——应当把它视为稀缺的工作记忆，精心管理
2. **可逆优先于有损**——先卸载，再裁剪，最后才摘要
3. **Context Rot 是真实威胁**——性能退化远在 token 上限之前就开始了
4. **没有银弹**——强大的 Agent 系统通常组合多种策略
5. **更大的窗口不能解决所有问题**——窗口大小有帮助，但无法替代精细的上下文管理
6. **上下文工程是 Agent 智能的运维层**——它决定 Agent 如何在任务变长、工具变嘈杂时保持连贯

## 参考资源

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic 官方上下文工程指南
- [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/en/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) — Manus 的上下文工程实践
- [Context Rot: How Increasing Input Tokens Impacts LLM Performance](https://research.trychroma.com/context-rot) — Chroma 的 Context Rot 研究
- [SWE-Pruner: Self-Adaptive Context Pruning for Coding Agents](https://arxiv.org/abs/2601.16746) — 面向编码 Agent 的自适应上下文裁剪
- [Galileo Evaluation Platform](https://galileo.ai/) — Agent 可观测性与评估指标
