# AI Agent 的上下文工程：构建 Manus 的经验教训

> 原文链接：[Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/zh-cn/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)
>
> 日期：2025-07-18
>
> 作者：Yichao 'Peak' Ji（Manus）
>
> 主题：agent, context-engineering, kv-cache, production
>
> 类型：article

---

## 一、上下文工程 vs 端到端训练

Manus 团队在项目初期面临抉择：用开源模型训练端到端 Agent 模型，还是基于前沿模型的上下文学习能力构建 Agent？Peak 从上一个创业公司的教训（自训 NLP 模型被 GPT-3 一夜淘汰）中得出结论：**押注上下文工程**。

核心优势：

- 几小时交付改进，而非几周
- 产品与底层模型保持正交——"如果模型进步是潮水，Manus 要做船，而不是固定在海底的柱子"

尽管如此，上下文工程绝非易事——Manus 已重建框架 **4 次**。团队将这种手动架构搜索、提示调整和经验猜测的过程称为"随机研究生下降"（Stochastic Graduate Descent）。

---

## 二、围绕 KV Cache 进行设计

KV Cache 命中率是 Agent 最重要的单一指标，直接影响延迟和成本。

| 指标 | 数据 |
|------|------|
| Manus 平均输入/输出 Token 比 | **100:1** |
| Claude Sonnet 缓存 vs 未缓存成本差 | **10 倍**（$0.30 vs $3.00/百万 Token） |

### 三条关键实践

1. **保持提示前缀稳定** — 不要在系统提示开头放精确到秒的时间戳，一个 token 的变化会使该 token 之后的所有缓存失效
2. **上下文只追加不修改** — 确保 JSON 序列化键顺序稳定，避免悄无声息地破坏缓存
3. **明确标记缓存断点** — 对于不支持自动增量前缀缓存的提供商，至少确保系统提示结尾有缓存断点

自托管场景（如 vLLM）：启用前缀/提示缓存，使用会话 ID 在分布式节点间一致路由请求。

---

## 三、遮蔽（Masking），而非移除

### 问题

Agent 通常定义了很多工具（如 `browser_click`、`shell_exec`、`message_reply` 等），这些工具定义序列化后位于上下文前部（系统提示前后）。在某个步骤，你可能只想让 Agent 使用浏览器工具。**直觉做法**是删掉不需要的工具定义——但这会使后续所有动作和观察的 KV Cache 全部失效，成本和延迟直接翻倍。

### 解决方案：响应预填充

保持工具定义始终不变，通过**预填充模型回复的开头 token** 来约束动作空间。LLM 生成回复是逐 token 输出的，如果你提前帮模型写好回复的前几个 token，模型就只能接着往下补全，无法选择被"堵死"的路径。

假设模型调用工具的输出格式为 `{"name": "browser_click", "arguments": {...}}`，三种预填充模式：

| 模式 | 预填充内容 | 效果 |
|------|-----------|------|
| **Auto** | `<\|im_start\|>assistant` | 模型自由决定是否调用函数 |
| **Required** | 预填充到工具调用令牌 | 必须调用函数，但哪个都行 |
| **Specified** | `{"name": "browser_` | 只能补全 `browser_` 开头的工具名 |

第三种就是"遮蔽"——你没有删掉任何工具定义，但通过写死回复前缀，让模型只能从 `browser_click`、`browser_navigate` 等工具中选择。等到该执行命令时，把前缀改成 `{"name": "shell_` 即可，**全程工具定义不变，KV Cache 完好无损**。

### 如何实现预填充

| 方式 | 做法 | 控制粒度 |
|------|------|----------|
| **Claude API** | 在 `messages` 末尾加一条未完成的 `assistant` 消息 | 前缀级（如 `browser_`） |
| **OpenAI API** | 使用 `tool_choice` 参数指定工具 | 单个工具级 |
| **自托管 (vLLM 等)** | 直接在 prompt 末尾拼接前缀，可配合约束解码 | 完全自由控制 |

### 设计技巧

工具名使用**一致前缀**（如 `browser_`、`shell_`），可无状态地约束到特定工具组，无需有状态的 logits 处理器。

---

## 四、文件系统作为终极上下文

128K Token 窗口在真实 Agent 场景中的三个痛点：

1. **观察结果可能非常庞大** — 网页、PDF 等非结构化数据容易超限
2. **性能随上下文长度下降** — 即使技术上支持该窗口大小
3. **长输入成本高昂** — 即使有前缀缓存，仍需为每个 token 的传输和预填充付费

Manus 的方案：**将文件系统视为终极上下文** — 大小不受限、天然持久化、Agent 可直接操作。模型学会按需写入和读取文件，将文件系统作为结构化的外部记忆。

### 可恢复的压缩策略

| 场景 | 可移除内容 | 保留内容 |
|------|-----------|----------|
| 网页 | 页面内容 | URL |
| 文档 | 文档内容 | 沙盒中的文件路径 |

核心原则：**任何压缩都必须是可恢复的**，因为无法预测哪个观察结果在十步之后可能变得至关重要。

> Peak 的前瞻思考：如果 SSM（状态空间模型）能掌握文件级记忆——将长期状态外部化而非保存在上下文中——那么 SSM 的速度和效率可能开启新一类 Agent，成为神经图灵机的真正继任者。

---

## 五、通过复述（Recitation）操控注意力

Manus 在处理复杂任务时会创建 `todo.md` 并逐步更新——这不是花哨的功能，而是**刻意的注意力操控机制**。

| 问题 | 解决方案 |
|------|----------|
| 平均 ~50 次工具调用，模型容易偏离目标 | 不断重写 todo 列表，将全局计划推入近期注意力范围 |
| "丢失在中间"（Lost in the Middle）效应 | 用自然语言让模型注意力偏向任务目标 |

无需特殊架构变更，通过自然语言自行调控注意力分配。

---

## 六、保留错误的内容

改善 Agent 行为最有效的方法之一：**将失败的尝试保留在上下文中**。

- 模型看到失败行动 + 错误观察/堆栈跟踪 → 隐式更新内部信念 → 降低重复错误概率
- **错误恢复是真正代理行为的最明显指标之一**
- 学术基准和公共评测中对此严重代表不足，通常只关注理想条件下的成功率

---

## 七、避免被少样本示例困住

当上下文充满类似的行动-观察对时，模型会盲目模仿模式。例如用 Manus 审查 20 份简历时，Agent 容易陷入重复节奏，导致偏离和过度泛化。

解决方法：**引入结构化变化（Controlled Randomness）**

- 不同的序列化模板
- 替代性措辞
- 顺序或格式上的微小噪音

> "你的上下文越单一，你的 Agent 就越脆弱。"

---

## 八、核心启示

1. **押注上下文工程而非模型训练** — 保持与底层模型正交，快速迭代
2. **KV Cache 命中率是最重要的指标** — 前缀稳定、只追加、标记断点
3. **遮蔽而非移除工具** — 通过预填充约束动作空间，不破坏缓存
4. **文件系统是终极上下文** — 无限容量、持久化、可恢复的压缩
5. **复述操控注意力** — todo.md 是将全局计划推入近期注意力的机制
6. **保留错误** — 失败尝试是 Agent 自我修正的关键上下文
7. **打破少样本模式** — 上下文多样性防止 Agent 僵化
8. **上下文工程是实验科学** — 没有银弹，只有"随机研究生下降"

---

## 参考资源

- [Manus 官方博客原文](https://manus.im/zh-cn/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)
- [In-context Learning 论文](https://arxiv.org/abs/2301.00234)
- [KV Caching Explained](https://medium.com/@joaolages/kv-caching-explained-276520203249)
- [vLLM Prefix Caching](https://docs.vllm.ai/en/stable/design/v1/prefix_caching.html)
- [Hermes Function Calling 格式](https://github.com/NousResearch/Hermes-Function-Calling)
- [Neural Turing Machines](https://arxiv.org/abs/1410.5401)
- [ReAct Agent 论文](https://arxiv.org/abs/2210.03629)
