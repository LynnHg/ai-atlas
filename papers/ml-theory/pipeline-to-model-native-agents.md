# 超越管道：从 Pipeline 到 Model-native 的 Agent AI 范式转变

> 原文链接：[Beyond Pipelines: A Survey of the Paradigm Shift toward Model-Native Agentic AI](https://arxiv.org/abs/2510.16720)
>
> 日期：2025-10-19
>
> 作者：Jitao Sang, Jinlin Xiao 等（北京交通大学）
>
> 主题：agent, model-native, reinforcement-learning, planning, tool-use, memory
>
> 类型：paper

---

## 一、核心论点：两大范式

### Pipeline-based（管道范式）

```
a = f_pipeline(π_θ)
```

LLM 是被外部逻辑编排的功能组件。规划、工具使用、记忆由外部手工逻辑控制。

- 规划：CoT、ToT 等 Prompt 技巧
- 工具：ReAct 式外部循环
- 记忆：对话摘要、RAG 检索

特点：模块化、可解释，但**刚性、脆弱**，无法适应未预见的情况。

### Model-native（模型原生范式）

```
π_agent ≡ π_θ
```

Agent 的策略就是模型的内部策略。能力内化到参数中。

- 规划：o1/R1 通过大规模 RL 学会自主推理
- 工具：o3/Kimi K2 将工具调用融入推理过程
- 记忆：MemoryLLM 将记忆参数化为前向传播的一部分

> 从"构建应用智能的系统"到"发展通过经验增长智能的模型"。

---

## 二、RL 是范式转变的引擎

### SFT vs RL

| SFT | RL |
|-----|-----|
| 模仿静态数据集 | 在动态环境中探索 |
| 需要人类标注完整轨迹（代价极高） | 只需结果奖励 |
| 无法发现新策略 | 能发现更优策略 |

### 统一训练范式：LLM + RL + Task

关键算法演进：

| 算法 | 贡献 |
|------|------|
| PPO/DPO + RLHF | 早期，适合单轮对齐 |
| **GRPO**（DeepSeek） | 去掉 Critic 网络，组内相对奖励 |
| **DAPO** | 解耦裁剪 + 动态采样，适合多轮长时 Agent |

---

## 三、三大能力的演进

### 规划

| Pipeline | Model-native |
|----------|-------------|
| 外部符号规划器（PDDL） | o1：RL 学会自主推理 |
| CoT/ToT Prompt | R1：仅结果奖励训练规划 |

### 工具使用

| Pipeline | Model-native |
|----------|-------------|
| 单轮 function calling | o3：工具调用融入推理 |
| ReAct 外部循环 | K2：合成轨迹 + 多阶段 RL |

### 记忆

| Pipeline | Model-native |
|----------|-------------|
| 对话摘要 + RAG | Qwen-2.5-1M：扩展原生上下文 |
| 外部向量数据库 | MemAct：上下文管理作为可学习工具 |
| 手动管理 | MemoryLLM：记忆参数化 |

---

## 四、两大应用演进

### Deep Research Agent

Pipeline → Model-native：从 Perplexity 工程化管道到 OpenAI Deep Research（基于 o3 微调）。

挑战：开放网络信息噪声 + 开放式研究任务的奖励定义。

### GUI Agent

Pipeline → Model-native：从 AppAgent（XML + LLM 编排）到 UI-TARS/GUI-Owl（端到端 RL）。

挑战：像素级精度 + GUI 环境非平稳性。

---

## 五、核心启示

1. **Agent AI 正经历 Pipeline → Model-native 的范式转变**——外部能力逐步内化到模型参数
2. **RL 是核心引擎**——从模仿到结果驱动的探索
3. **SFT 的根本限制是过程数据短缺**——人类无法标注复杂 Agent 任务的完整轨迹
4. **GRPO/DAPO 使长时 Agent 训练成为可能**
5. **统一范式 LLM + RL + Task**——适用于语言、视觉和具身域
6. **当前上下文工程属于 Pipeline 范式**——是当下最佳实践，但未来可能被 Model-native 取代
7. **两种范式将长期共存**——Model-native 逐步内化更多能力，Pipeline 在近期仍是实用方案

---

## 参考资源

- [论文全文 (arXiv)](https://arxiv.org/abs/2510.16720)
- [论文 HTML 版](https://arxiv.org/html/2510.16720v2)
- [论文引用列表 (GitHub)](https://github.com/ADaM-BJTU/model-native-agentic-ai)
- [DeepSeek R1 技术报告](https://arxiv.org/abs/2501.12948)
- [GRPO 论文](https://arxiv.org/abs/2402.03300)
