# LLM Agent 漫游指南：从零构建编码 Agent 的实战经验

> 原文链接：[The Hitchhikers Guide to LLM Agent](https://saurabhalone.com/blog/agent)
>
> 日期：2026-01-03
>
> 作者：Saurabh Alone（Hakken 作者）
>
> 主题：agent, context-engineering, memory, sub-agent, kv-cache, evaluation
>
> 类型：article

---

## 一、核心层级

作者的优先级排序：

1. **上下文工程** — 掌握这个否则其他都会失败
2. **评估** — 先建评估，用数据迭代
3. **掌控技术栈** — 不要外包给框架
4. **简单模式** — 用已证明有效的方案
5. **记忆** — 只在真正需要时才加

---

## 二、上下文工程 8 个实战技巧

### 2.1 简洁系统提示词

强模型（Opus 4.5）只需最简提示词。长提示词更适合便宜模型。

### 2.2 精简工具集

用 Bash 处理大部分文件操作（ls/grep/find），省大量 Token。Vercel 移除了 80% 的 Agent 工具后效果更好。

### 2.3 80% 阈值压缩

上下文超 80% 时自动摘要，保留系统提示 + 摘要 + 最近 5 条消息。保留关键决策、错误和待办。减少 **35-40%** 上下文使用。

> 调试/审查时不要压缩。50k Token 以下不需要压缩。

### 2.4 激进清理旧工具结果

每 10 次工具调用清理旧结果（保留最近 5 个），替换为 `[Result cleared]`。

### 2.5 KV-Cache 优化三规则

| 规则 | 说明 |
|------|------|
| 保持 prompt 前缀稳定 | 系统提示词不要放时间戳！一个 Token 变化让后续 Cache 全部失效 |
| 上下文只追加不修改 | 修改中间消息打破该点后的 Cache |
| 确定性序列化 | `json.dumps(data, sort_keys=True)` |

> Sonnet 4.5 缓存 $0.30/MTok vs 未缓存 $3/MTok — **10 倍差异**。

### 2.6 结构化笔记（todo.md）

- **持久记忆**：关键上下文活在上下文窗口外
- **注意力操控**：不断重写 todo 将目标推入上下文末尾，利用 U 型注意力模式

### 2.7 渐进式发现

不要预加载所有内容，让 Agent 自己探索和按需加载。

### 2.8 短会话优于长会话

200k Token 就够。拆分工作为小线程，每任务一个线程。

---

## 三、掌控 Prompt 和控制流

> "很多框架里你做 `Agent(role="...")` 然后完全不知道发送了什么 Token 给模型。"

- 掌控每一个 Token——出问题能调试
- 掌控 Agent 循环——能中断、能处理边缘情况

### 信任模式

全权限给 Agent 效果反而更好：
- 频繁权限请求**打断思维链**
- 全权限下 Agent 能多步向前规划，更有策略性
- 类似给信任的人类助手充分授权

---

## 四、记忆：何时需要

| 应用类型 | 是否需要 | 原因 |
|----------|---------|------|
| **水平应用**（通用聊天） | 不太需要 | 每次对话不同，好的上下文工程就够 |
| **垂直应用**（编码、理财、销售） | 需要 | 同一领域反复工作，记住偏好有价值 |

> "构建可靠 Agent 最重要的是上下文工程。记忆只在真正需要时才加。"

简单文件存储（JSON）足以应对大多数场景，不需要向量数据库。

---

## 五、子 Agent 模式

适合：深度研究（并行探索）、代码审查（非相互依赖）、顺序任务。

不适合：并行相互依赖的任务——子 Agent 之间不通信，集成会失败。

---

## 六、Context Rot 数据

Chroma 对 18 个模型的测试：
- 100k Token 时比 10k Token 时准确率下降 **15-30%**
- 连贯文本反而比打乱的随机文本导致更差的检索
- GPT-5.1 和 Claude 4 仍受此影响

---

## 七、核心启示

1. **上下文工程是一切**——其他优化都建立在此基础上
2. **KV-Cache 优化有 10 倍成本差异**——前缀稳定、只追加、键排序一致
3. **掌控你的 prompt 和控制流**——不要让框架变成黑盒
4. **80% 阈值压缩 + 每 10 次清理旧工具结果**——实用的管理公式
5. **记忆对垂直应用有价值，对水平应用是过度工程**
6. **全权限比频繁审批效果更好**——权限请求打断思维链
7. **短会话优于长会话**——200k Token 就够
8. **Skills > MCP**——Markdown 文件比 MCP Server 更省 Token

---

## 参考资源

- [Hakken 开源项目](https://github.com/saurabhaloneai/hakken)
- [12-Factor Agents](https://github.com/humanlayer/12-factor-agents)
- [Context Rot 研究](https://research.trychroma.com/context-rot)
- [Lost in the Middle 论文](https://arxiv.org/abs/2307.03172)
- [Vercel: We removed 80% of our agent's tools](https://vercel.com/blog/we-removed-80-percent-of-our-agents-tools)
- [200k Tokens Is Plenty](https://ampcode.com/200k-tokens-is-plenty)
- [Hamel Husain: Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/)
- [Cognition: Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents)
- [Simon Willison: Claude Skills > MCP](https://simonwillison.net/2025/Oct/16/claude-skills/)
