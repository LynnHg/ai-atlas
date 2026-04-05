# rLLM SDK：无需改代码即可训练任何 Agent 程序

> 原文链接：[rLLM SDK: Training Any Agentic Program without Code Changes](https://rllm-project.com/post.html?post=sdk.md)
>
> 日期：2025-12-10
>
> 作者：Tianhao Wu, Sijun Tan, rLLM Team
>
> 主题：fine-tuning, agent, reinforcement-learning, rl-training
>
> 类型：article

---

## 一、核心问题：Agent 工程与 RL 训练的语言鸿沟

| Agent 工程 | RL 训练 |
|-----------|---------|
| 语言：**API 调用** | 语言：**Token** |
| `client.chat.completions.create()` | Token IDs `[1, 234, 567]` |
| 关注：消息和补全 | 关注：Token 概率和轨迹 |
| 框架：LangGraph、AutoGen、Claude SDK | 框架：VERL、TRL |

### 传统桥接方式的两大问题

| 问题 | 后果 |
|------|------|
| **代码重复** | 维护两个版本的 Agent（生产版 + 训练版），逻辑差异导致训练的不是部署的 Agent |
| **重新分词不匹配** | 分词器上下文敏感，重新分词可能产生不同 Token 序列，导致 off-policy 训练和**奖励坍缩** |

> 案例：`" Pantom"` 被分词为 `[53122, 316]` 或 `[393, 30002]`——同一词，不同 Token 序列。

---

## 二、rLLM SDK 的解决方案

核心理念：所有 Agent 框架都建立在 API 调用之上，**在 API 层拦截即可自动完成训练数据转换**。

### 架构三组件

| 组件 | 作用 |
|------|------|
| **LiteLLM Proxy** | 拦截 API 调用，路由到推理引擎，捕获模型实际生成的 Token IDs 和 log 概率 |
| **SQLite Store** | 持久存储 Token、概率和元数据——训练数据的唯一真实来源 |
| **Agent Trainer（基于 VERL）** | 从 Store 获取数据转换为轨迹，执行 RL 训练 |

### 工作流程

```
你的 Agent 代码（LangGraph / AutoGen / Claude SDK）
  → 仅需替换 client → rLLM SDK 的 get_chat_client()
    → LiteLLM Proxy 拦截 API 调用
      → 路由到 vLLM 推理引擎
        → 捕获精确 Token IDs + log 概率
          → 存储到 SQLite
            → Agent Trainer 获取数据进行 RL 训练
```

> 关键优势：使用模型**实际生成的 Token** 进行训练，保证梯度更新是 on-policy 的。"你构建的就是你训练的。"

---

## 三、渐进式教程

| 级别 | 内容 | 学习重点 |
|------|------|----------|
| 初级 | 简单数学 Agent | `get_chat_client()`、单步 rollout、trainer 设置 |
| 中级 | Solver-Judge 工作流 | `@trajectory` 装饰器、多 Agent 组合、逐步奖励 |
| 高级 | LangGraph RAG | LangChain 集成、多轮 tracing、工具使用 Agent |

---

## 四、核心启示

1. **Agent 工程和 RL 训练之间存在"语言鸿沟"**——API 调用 vs Token 序列
2. **重新分词是被低估的危险**——同一文本重新分词可能产生不同 Token 序列
3. **在 API 层拦截是优雅的解决方案**——所有框架都建立在 API 调用之上
4. **"你构建的就是你训练的"**——消除生产代码和训练代码的差异
5. **RL 训练正在从研究团队特权走向工程师日常工具**——民主化趋势
6. **Agent 未来不仅使用工具，还能通过 RL 持续改进自己**

---

## 参考资源

- [rLLM 项目主页](https://rllm-project.com/)
- [VERL：分布式 RL 训练](https://github.com/volcengine/verl)
- [LiteLLM Proxy](https://docs.litellm.ai/docs/proxy/quick_start)
- [重新分词问题详解](https://wandb.ai/tianhaowu/rllm-agent/reports/Tokenization-Mismatch-in-Text-Level-Operations--VmlldzoxNDg0MTcwMw)
