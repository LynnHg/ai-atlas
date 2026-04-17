# LLM Agent 的外化理论：记忆、技能、协议与 Harness 工程统一综述

> 原文链接：[Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering](https://arxiv.org/html/2604.08224v1)
>
> 日期：2026-04-09
>
> 作者：Chenyu Zhou, Huacan Chai 等（上海交通大学、中山大学、Carnegie Mellon University、上海创新研究院、OPPO）
>
> 主题：agent, memory, skills, protocols, harness-engineering
>
> 类型：paper

---

## 核心论点

这篇综述提出了一个统一理论框架：**外化（Externalization）** 是 LLM Agent 架构演进的核心逻辑。

论文的中心论断：Agent 的进步不仅来自更强的模型，更来自对认知负担的系统性外化——把原本需要模型内部完成的任务，搬移到外部持久、可检索、可复用的结构中，从而让模型更可靠地运作。

这一逻辑来自 Norman 的**认知人工物（Cognitive Artifacts）**理论：外部辅助工具不只是放大内部能力，而是**重构任务本身**，把难以直接完成的问题转化成模型擅长处理的形式。

---

## 一、从权重到上下文到 Harness：三个演进阶段

### 1.1 阶段一：能力在权重中（Weights）

- 模型能力 = 参数规模 + 预训练数据
- 优点：无需外部查询，快速推理，泛化能力强
- **核心局限**：知识与程序耦合在静态参数中，难以更新、审计、个性化

### 1.2 阶段二：能力在上下文中（Context）

- Few-shot、CoT、RAG 等技术：无需改权重，通过提示词改变行为
- 核心转变：**recall → recognition**（召回 → 识别）
- **核心局限**：上下文有限、昂贵、"中间丢失"问题、会话结束即遗忘

### 1.3 阶段三：能力通过基础设施实现（Harness）

- Agent 被持久记忆、工具注册表、协议定义、沙箱、子 Agent 编排、评估器所包围
- 代表系统：Auto-GPT、BabyAGI → AutoGen、MetaGPT → SWE-agent、OpenHands、Claude Code

| 层次 | 核心转变 | 代表技术 |
|------|----------|----------|
| Weights | 知识内化 | GPT-4, DeepSeek, Qwen |
| Context | recall → recognition | RAG, CoT, Few-shot |
| Harness | improvisation → composition | Memory Store, Skills, Protocols |

---

## 二、三种外化维度

### 2.1 记忆外化（Memory）：跨时间的状态持久化

**核心转变**：从内部回忆问题 → 外部检索识别问题

#### 外化的内容（四类状态）

| 记忆类型 | 内容 | 典型实现 |
|----------|------|----------|
| **工作上下文（Working Context）** | 当前任务中间状态、草稿、变量 | OpenHands 工作区文件 |
| **情节记忆（Episodic）** | 历史决策、工具调用、失败记录 | Reflexion 反思摘要 |
| **语义知识（Semantic）** | 领域事实、抽象规律、通用规则 | RAG 知识库 |
| **个性化记忆（Personalized）** | 用户偏好、习惯、跨会话偏好 | VARS、IFRAgent |

#### 记忆架构的四个演进范式

1. **单块上下文（Monolithic Context）**：历史直接放 prompt，简单但上限低
2. **检索存储（Retrieval Storage）**：GraphRAG、ENGRAM、SYNAPSE 等，解决容量问题
3. **层级记忆（Hierarchical）**：Mem0、MemGPT、MemoryOS，实现提取/合并/遗忘三元生命周期
4. **自适应记忆（Adaptive）**：MemEvolve、MemRL，用 RL/MoE 动态优化检索策略

> **关键洞见**：检索质量比存储容量更重要。评价记忆的标准不是"存了多少"，而是"当前决策是否清晰可读"。

---

### 2.2 技能外化（Skills）：程序性专业知识的复用

**核心转变**：从每次重新生成工作流 → 加载并遵循预定义程序

#### 技能包含的三个组件

| 组件 | 说明 |
|------|------|
| **操作程序（Operational Procedure）** | 任务分解步骤、依赖、终止条件 |
| **决策启发（Decision Heuristics）** | 分支点的默认选择、回退规则 |
| **规范约束（Normative Constraints）** | 合规边界、访问限制、测试要求 |

#### 技能演化的三个阶段

- **Stage 1**：原子执行原语（Toolformer 等工具调用）
- **Stage 2**：大规模原语选择（Gorilla、ToolLLM 等工具检索）
- **Stage 3**：**技能即封装的专业知识**（SOP-guided、Voyager Skills、计算机操作技能包）

#### 技能的五个运行时要素

```
Specification（规格说明）
    ↓
Discovery（注册与检索）
    ↓
Progressive Disclosure（渐进披露：名称 → 摘要 → 完整指南）
    ↓
Execution Binding（与工具/协议绑定）
    ↓
Composition（串行/并行/条件路由/递归组合）
```

#### 技能的四种获取方式

| 获取方式 | 说明 |
|----------|------|
| **Authored（人工编写）** | SKILL.md、AGENTS.md、SOP 模板 |
| **Distilled（从轨迹提炼）** | 从成功执行历史抽象出可复用程序 |
| **Discovered（自主发现）** | Voyager：探索→执行→自验证→课程，持续增长技能库 |
| **Composed（组合生成）** | 把多个低层技能打包成高层新技能 |

#### 技能的边界条件（失效场景）

- **语义对齐问题**：描述与实际用途不符（SkillProbe 发现大量 marketplace 中存在此问题）
- **可移植性与陈旧性**：环境变更后技能失效
- **不安全组合**：孤立安全的技能组合后可能有安全漏洞（prompt injection、数据外泄）
- **上下文依赖退化**：详细技能指南可能淹没全局任务追踪

---

### 2.3 协议外化（Protocols）：交互结构的治理

**核心转变**：从 ad-hoc 即兴协调 → 结构化、可治理的交互契约

#### 协议外化的四个维度

| 维度 | 说明 |
|------|------|
| **调用语法（Invocation Grammar）** | 参数名、类型、返回结构 |
| **生命周期语义（Lifecycle Semantics）** | 状态转移、任务完成/失败规则 |
| **权限与信任边界（Permission & Trust）** | 访问控制、数据流向、可审计性 |
| **发现元数据（Discovery Metadata）** | 注册表、能力卡片、schema 端点 |

#### 协议分类总览

| 类别 | 代表协议 | 核心作用 |
|------|----------|----------|
| **Agent-Tool** | **MCP**（Anthropic）、ToolUniverse | 标准化工具发现与调用 |
| **Agent-Agent** | **A2A**（Google）、ACP（IBM）、ANP | 标准化委托、状态交换、协商 |
| **Agent-User** | **A2UI**（Google）、AG-UI（CopilotKit）| UI 结构生成 + 实时状态流 |
| **垂直领域** | UCP（电商）、AP2（支付） | 高风险工作流的专项治理 |

> **MCP 的地位**：工具-Agent 协议中最清晰的代表，通过 JSON-RPC 2.0 实现动态能力发现与标准化调用，使工具生态从点对点适配变为协议化集成。

---

## 三、Harness Engineering：统一外化的认知环境

### 3.1 什么是 Harness？

Harness 不是"模型 + 外挂组件"，而是 **Agent 运行的认知环境**。它决定了模型感知什么、记住什么、能调用什么、被允许做什么。

> "一个实用的 Agent 更应被理解为**在 Harness 中运行的模型**，而不是附带外围能力的模型。"

### 3.2 Harness 的六个分析维度

```
┌─────────────────────────────────────────────────────┐
│                    Foundation Model                  │
│                     (Agent Core)                     │
├─────────────────┬────────────────┬───────────────────┤
│  Memory Module  │  Skills Module │ Protocols Module  │
│ (状态持久化)    │ (程序性专业知识)│ (交互结构治理)    │
├─────────────────┴────────────────┴───────────────────┤
│           Harness Operational Surfaces               │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │Permission│  │ Control  │  │  Observability   │   │
│  │(沙箱隔离)│  │(循环控制)│  │(结构化追踪与反馈)│   │
│  └──────────┘  └──────────┘  └──────────────────┘   │
└─────────────────────────────────────────────────────┘
```

| 维度 | 说明 | 实践例子 |
|------|------|----------|
| **Agent 循环与控制流** | perceive→retrieve→plan→act→observe 循环；步数、递归深度、成本上限 | Claude Code 的执行循环 |
| **沙箱与执行隔离** | 文件系统快照、网络限制、资源配额 | Codex 云沙箱、Claude Code 权限模式 |
| **人工监督与审批门** | 预执行审批、执行后审查、风险触发升级、Hook 系统 | 可配置的自主程度参数 |
| **可观测性与结构化反馈** | 模型调用日志、工具调用链、决策分支追踪 | 调试、合规审计、失败模式识别 |
| **配置、权限与策略编码** | 用户级/项目级/组织级分层配置 | 同一 Agent 在不同部署策略下运行 |
| **上下文预算管理** | 摘要压缩 + 优先级驱逐 + 渐进加载 | 规划阶段多记忆，执行阶段多技能 |

---

## 四、模块间的交互（Cross-Cutting Analysis）

### 4.1 六条核心耦合关系

```
Memory ──distill──→ Skills    (经验提炼为可复用程序)
Skills ──record──→ Memory     (执行轨迹写回为记忆)
Skills ──invoke──→ Protocols  (抽象程序通过协议落地为动作)
Protocols ──generate──→ Skills (稳定接口催生领域技能包)
Memory ──route──→ Protocols   (历史经验影响协议路径选择)
Protocols ──assimilate──→ Memory (工具输出规范化后入库)
```

### 4.2 从模型 I/O 视角看三个模块

| 模块 | 在模型边界的角色 | 失效类型 |
|------|-----------------|----------|
| Memory | **上下文输入**：选择性注入历史与情境 | 检索错误 → 推理基于错误上下文 |
| Skills | **指令输入**：按需加载程序性指南 | 程序指导错误/不匹配 |
| Protocols | **输出约束**：将生成限制在 JSON schema 内 | 动作 schema 违反接口契约 |

### 4.3 参数化 vs 外化：如何分配？

| 决策维度 | 倾向外化 | 倾向参数化 |
|----------|----------|------------|
| **更新频率** | 快速变化的知识（API、组织结构） | 稳定的背景能力（语言理解、推理） |
| **复用性** | 跨任务/跨 Agent 频繁复用 | 一次性或高度特定的行为 |
| **可审计性** | 高风险、需要合规检查 | 低风险的通用推理 |
| **延迟要求** | 可接受额外检索开销 | 超低延迟的纯语义任务 |

---

## 五、未来方向

### 5.1 外化边界的扩展

- **规划对象化**：计划从 in-context 推理产物 → 持久、可检索、可跨 Agent 共享的 Harness 对象
- **评估外化**：把评估标准、rubric、验证程序作为运行时 Harness 组件（而非事后评测）
- **编排逻辑的元外化**：Harness 自身的配置、策略变成可被 Agent 检查和修改的对象 → 自进化 Harness

### 5.2 具身 Agent 的相同逻辑

数字 Agent 与机器人 Agent 面临相同的分裂：

| 角色 | 数字 Agent | 机器人 Agent |
|------|-----------|-------------|
| **大脑（Cerebrum）** | LLM 规划器 | 高层 LLM Agent |
| **小脑（Cerebellum）** | 工具调用、代码解释器 | VLA 运动技能模块 |
| **调度层** | Harness | 同样是 Harness 模式 |

### 5.3 自进化 Harness

未来 Harness 目标：不仅托管外化模块，还能根据执行反馈自动调整自身策略——当 Harness 配置本身成为外化对象时，Agent 系统就能自我优化。

### 5.4 治理与风险

- **认知开销**：外化增加检索延迟、上下文竞争、协调复杂度
- **安全完整性**：技能文件成为 prompt injection 攻击面；"致命三联征"（敏感数据 + 无约束外部通信 + 未经验证执行）
- **基础设施治理**：共享外化资源需要读写权限、冲突解决、版本管理

---

## 核心启示

1. **外化是 Agent 可靠性的根本机制**：可靠的 Agent 不是更大的模型，而是更好地把认知负担外化到持久结构中的系统。

2. **记忆 = 时间维度的状态外化**：评价记忆的标准不是容量，而是"检索后是否让当前决策更清晰"。

3. **技能 = 程序性专业知识的复用化**：从即兴生成工作流 → 加载并遵循预定义程序，是减少 Agent 行为方差的关键。

4. **协议 = 交互结构的治理化**：MCP、A2A 等协议不只是工程管道，而是把模糊的 ad-hoc 协调变为可审计的结构化契约。

5. **Harness 是认知环境，不只是基础设施**：它决定模型"看到什么、记住什么、能做什么"——Agent 的智能分布在模型与 Harness 中，不在模型单独。

6. **三模块协作优于单独优化**：Memory→Skills→Protocols 形成正向反馈循环，断掉任何一环都会降低整体可靠性。

7. **技能文件是安全攻击面**：SKILL.md 等技能文件本身可成为 prompt injection 向量，需要在 Harness 层做验证和权限控制。

8. **外化边界会动态移动**：随着模型更强和基础设施更成熟，什么应该外化、什么保留在参数中的最优分割点会持续变化。

---

## 参考资源

- [原论文](https://arxiv.org/html/2604.08224v1) — arXiv:2604.08224v1
- [MCP 协议文档](https://modelcontextprotocol.io/) — Anthropic Model Context Protocol
- [A2A 协议](https://github.com/google/A2A) — Google Agent-to-Agent Protocol
- [AG-UI 协议](https://github.com/copilotkit/agui) — CopilotKit Agent-User Protocol
- [CoALA 框架](https://arxiv.org/abs/2309.02427) — Cognitive Architectures for Language Agents（本文引用最多的相关工作）
- [Voyager](https://arxiv.org/abs/2305.16291) — 自动发现技能的典型案例
- [MemGPT](https://arxiv.org/abs/2310.08560) — 层级记忆架构代表
