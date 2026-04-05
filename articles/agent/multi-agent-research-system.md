# Anthropic 多 Agent 研究系统：架构设计与工程实践

> 原文链接：[How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
>
> 日期：2025-06-13
>
> 作者：Jeremy Hadfield, Barry Zhang, Kenneth Lien, Florian Scholz, Jeremy Fox, Daniel Ford
>
> 主题：agent, multi-agent, orchestrator-worker, prompt-engineering, evaluation
>
> 类型：article

---

## 一、为什么选择多 Agent 架构

### 1.1 研究任务的特殊性

- 研究是开放式问题，无法预先规划固定路径
- 需要根据中间发现动态调整方向，天然适合 Agent
- 线性的一次性管道（one-shot pipeline）无法胜任

### 1.2 多 Agent 的核心价值——"压缩"

- 子 Agent 各自拥有独立上下文窗口，并行探索不同方面
- 将最重要的信息浓缩后返回主 Agent
- 每个子 Agent 关注点分离（separation of concerns），减少路径依赖

### 1.3 关键性能数据

| 指标 | 数据 |
|------|------|
| 多 Agent vs 单 Agent 性能提升 | **90.2%**（Opus 4 主 Agent + Sonnet 4 子 Agent vs 单 Agent Opus 4） |
| Token 用量对性能差异的解释度 | **80%**（另外 20% 来自工具调用次数和模型选择） |
| Agent 对比 Chat 的 Token 消耗 | 单 Agent 约 **4x**，多 Agent 约 **15x** |
| 并行搜索提速 | 复杂查询研究时间减少高达 **90%** |

### 1.4 适用场景

**适合**：高价值、高度可并行、信息量超出单上下文窗口、需要对接大量复杂工具的任务

**不适合**：所有 Agent 需要共享同一上下文、Agent 间存在大量依赖的任务（如大多数编码任务）

---

## 二、系统架构：Orchestrator-Worker 模式

### 2.1 整体流程

```
用户查询
  → LeadResearcher（主 Agent）分析查询、制定策略
    → 保存计划到 Memory（防止上下文截断时丢失）
    → 并行创建 N 个 Subagent，分配具体研究子任务
      → 每个 Subagent 独立执行：Web 搜索 → Interleaved Thinking 评估 → 返回发现
    → LeadResearcher 综合结果，决定是否需要更多研究
      → 如需要 → 创建新一轮 Subagent
      → 如充分 → 退出研究循环
  → 交给 CitationAgent 处理引用
  → 返回带引用的最终研究报告
```

### 2.2 与传统 RAG 的区别

| 特性 | 传统 RAG | 多 Agent 研究系统 |
|------|----------|-------------------|
| 检索方式 | 静态检索（一次性获取相似 chunk） | 多步动态搜索，根据发现自适应调整 |
| 探索深度 | 固定 | 动态，可按需深入或拓宽 |
| 上下文管理 | 单一窗口 | 分布式上下文（每个子 Agent 独立窗口） |

---

## 三、Prompt 工程 8 条核心原则

### 3.1 像你的 Agent 一样思考

- 在 Console 中搭建仿真环境（使用完全相同的 Prompt 和工具）
- 逐步观察 Agent 行为，建立准确的心智模型
- 典型问题：Agent 已有足够结果却继续搜索、搜索查询过于冗长、选择了错误的工具

### 3.2 教主 Agent 如何委派任务

子 Agent 需要的完整上下文：
- **明确的目标**：具体的研究问题
- **输出格式**：期望的返回结构
- **可用工具和数据源**：指定使用哪些工具
- **任务边界**：与其他子 Agent 的分工

> 反面案例：模糊指令如"研究半导体短缺"导致子 Agent 重复劳动——一个探索 2021 汽车芯片危机，另外两个重复调查 2025 供应链。

### 3.3 按查询复杂度缩放工作量

在 Prompt 中嵌入明确的缩放规则：

| 查询类型 | Agent 数量 | 工具调用次数 |
|----------|-----------|-------------|
| 简单事实查找 | 1 | 3-10 |
| 直接对比 | 2-4 | 每个 10-15 |
| 复杂研究 | 10+ | 明确分工 |

> 早期版本常见问题：简单查询也启动大量子 Agent，造成资源浪费。

### 3.4 工具设计和选择至关重要

- Agent-工具接口的重要性等同于人机交互接口
- 每个工具需要**明确的用途**和**清晰的描述**
- 给 Agent 明确的工具选择启发式规则：
  - 先检查所有可用工具
  - 根据用户意图匹配工具
  - 用 Web 搜索做广泛外部探索
  - 优先使用专用工具而非通用工具
- MCP Server 让问题更复杂：Agent 会遇到从未见过的工具，描述质量参差不齐

### 3.5 让 Agent 自我改进

- Claude 4 模型能诊断 Prompt 失败原因并建议改进
- 实践：创建"工具测试 Agent"——给定有缺陷的 MCP 工具，Agent 尝试使用并重写工具描述
- 通过数十次测试发现关键细节和 Bug
- 成效：使用改进后的工具描述，后续 Agent 的任务完成时间**降低 40%**

### 3.6 先广后窄

- 搜索策略模拟专家人类行为
- Agent 常见错误：默认使用过长、过于具体的查询，导致结果很少
- 正确策略：短而广的查询 → 评估可用信息 → 逐步缩小聚焦

### 3.7 引导思考过程

- **Extended Thinking**：作为可控的思考草稿本
  - 主 Agent：规划方法、评估工具适配、确定查询复杂度和子 Agent 数量、定义分工
  - 子 Agent：使用 **Interleaved Thinking**，在工具结果后评估质量、识别缺口、优化下一次查询
- 测试表明 Extended Thinking 改善了指令遵循、推理能力和效率

### 3.8 并行工具调用

两层并行化：
1. 主 Agent 并行启动 3-5 个子 Agent（而非串行）
2. 每个子 Agent 并行调用 3+ 个工具

> 核心理念：Prompt 不是死板的规则，而是**协作框架**——定义分工、问题解决方法和工作量预算。

---

## 四、评估方法论

### 4.1 立即用小样本开始

- 不要等有数百个测试用例才开始评估
- 早期改动效果显著（如 30% → 80%），**20 个代表性查询**就能看出差异
- 小规模评估立即开始 > 完美评估推迟开始

### 4.2 LLM-as-Judge

- 用**单次 LLM 调用**按评分标准打分（0.0-1.0 + pass/fail）
- 比多个 Judge 更一致、更贴合人类判断
- 评分维度：

| 维度 | 说明 |
|------|------|
| 事实准确性 | 声明是否与来源匹配 |
| 引用准确性 | 引用的来源是否支持声明 |
| 完整性 | 是否覆盖所有请求的方面 |
| 来源质量 | 是否优先使用权威一手来源 |
| 工具效率 | 是否合理使用正确的工具 |

### 4.3 人工测试不可替代

- 发现自动评估难以捕捉的边缘情况
- 案例：早期 Agent 偏爱 SEO 内容农场，忽略权威学术 PDF 和个人博客
- 修复：在 Prompt 中添加来源质量启发式规则

### 4.4 关注终态而非过程

- 评估 Agent 是否达到**正确的最终状态**，而非是否走了"正确"路径
- 对复杂工作流，设置离散检查点验证关键状态变化
- Agent 可能走不同路径到达相同目标，这是合理的

### 4.5 涌现行为

- 多 Agent 系统存在涌现行为——对主 Agent 的微小改动可能不可预测地改变子 Agent 行为
- 需要理解交互模式，而非仅关注单个 Agent 行为

---

## 五、生产环境工程挑战

### 5.1 Agent 有状态且错误会级联

- Agent 长时间运行，跨多次工具调用维护状态
- 传统软件的小 Bug 对 Agent 可能是灾难性的
- 应对策略：
  - 建立**检查点和恢复机制**，不从头重启
  - 让模型感知工具失败并自适应（效果出奇地好）
  - 结合 AI 适应性 + 确定性保障（重试逻辑、定期检查点）

### 5.2 调试需要新方法

- Agent 决策是动态的、非确定性的，传统调试方法不够用
- 关键措施：**全链路生产级 Tracing**
- 监控 Agent 决策模式和交互结构（不监控对话内容以保护隐私）
- 帮助诊断根因、发现意外行为、修复常见故障

### 5.3 部署需要仔细协调

- Agent 系统是高度有状态的 Prompt + 工具 + 执行逻辑组合，几乎持续运行
- 更新部署时，Agent 可能处于任何执行阶段
- 解决方案：**Rainbow Deployment（彩虹部署）**——新旧版本同时运行，逐步切换流量

### 5.4 同步执行造成瓶颈

- 当前：主 Agent 同步等待子 Agent 完成，无法中途调度或协调
- 未来方向：异步执行——Agent 并发工作、按需创建新子 Agent
- 挑战：结果协调、状态一致性、错误传播

---

## 六、附录：实用技巧

### 6.1 长对话上下文管理

- Agent 在完成一个阶段后**总结并存储到外部 Memory**
- 上下文超过 200K Token 时会被截断，计划等关键信息需提前持久化
- 临近上下文限制时，启动新子 Agent（干净上下文），通过 Memory 继承必要信息

### 6.2 子 Agent 输出到文件系统

- 避免"传话游戏"（信息逐层传递导致失真）
- 子 Agent 将结构化成果（代码、报告、数据可视化）直接写入外部存储
- 只传**轻量引用**回主 Agent
- 减少 Token 开销，防止多阶段处理中的信息丢失

### 6.3 终态评估策略

- 评估会修改持久状态的 Agent 时，关注最终状态而非逐轮分析
- 对复杂工作流，拆分为离散检查点，在关键节点验证状态变化

---

## 七、核心启示

1. **Token 是核心资源**——多 Agent 的优势本质上来自于更聪明地使用更多 Token
2. **Prompt 是协作协议**——不是控制指令，而是定义分工和决策框架
3. **工具描述的质量直接决定 Agent 能力的上限**——糟糕的描述会让 Agent 彻底走偏
4. **小规模评估立即开始 > 完美评估推迟开始**——20 个用例就够看出效果
5. **从原型到生产的距离比想象中大得多**——错误会级联，状态管理是核心难题
6. **先广后窄的搜索策略**是提升 Agent 搜索质量的关键启发式规则
7. **让 Agent 自我改进**——用 Agent 测试和优化工具描述，形成正向循环
8. **并行化是性能倍增器**——两层并行（Agent 级 + 工具级）可将研究时间减少 90%

---

## 参考资源

- [Anthropic Cookbook - Agent Patterns & Workflows](https://platform.claude.com/cookbook/patterns-agents-basic-workflows)
- [Extended Thinking 文档](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)
- [Interleaved Thinking 文档](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#interleaved-thinking)
- [MCP 协议介绍](https://modelcontextprotocol.io/introduction)
- [BrowseComp 评估](https://openai.com/index/browsecomp/)
- [Rainbow Deployment 介绍](https://brandon.dimcheff.com/2018/02/rainbow-deploys-with-kubernetes/)
- [Clio 研究](https://www.anthropic.com/research/clio)
