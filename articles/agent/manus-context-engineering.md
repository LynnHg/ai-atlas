# Manus 的上下文工程实践：Reduce / Offload / Isolate

> 原文链接：[Context Engineering in Manus](https://rlancemartin.github.io/2025/10/15/manus/)
>
> 日期：2025-10-15
>
> 作者：Lance Martin（LangChain）
>
> 主题：agent, context-engineering, multi-agent, manus, production
>
> 类型：article

---

## 一、背景

Manus 是最受欢迎的通用消费级 Agent 之一。典型任务使用 **50 次工具调用**，每个会话运行在专用的云端虚拟机（E2B 沙箱）中。三大上下文工程策略：**Reduce / Offload / Isolate**。

---

## 二、Context Reduction（上下文精简）

### 工具结果的双重表示

| 版本 | 内容 | 存放位置 |
|------|------|----------|
| **Full（完整版）** | 原始工具调用结果 | 文件系统 |
| **Compact（精简版）** | 指向完整结果的引用（如文件路径） | 上下文窗口 |

策略：
- **旧的**工具结果 → Compact 版本替换（节省 Token，仍可按需获取完整版）
- **新的**工具结果 → 保持 Full 版本（指导下一步决策）

> 与 Anthropic 的 Context Editing 一致：自动清理过时的工具调用和结果。

### 摘要化

Compaction 到达边际递减后，对整个轨迹进行摘要：
- 使用**完整工具结果**生成摘要（不是精简版）
- 用 **Schema 定义摘要字段**，保证一致性

---

## 三、Context Isolation（上下文隔离）

### 多 Agent 架构

Manus 对多 Agent 持务实态度——不按人类角色分工，以**隔离上下文**为主要目标。

| 角色 | 职责 |
|------|------|
| **Planner** | 分配任务 |
| **Knowledge Manager** | 审查对话，决定哪些信息保存到文件系统 |
| **Executor 子 Agent** | 执行 Planner 分配的任务 |

### 从 todo.md 到 Planner Agent

Manus 最初用 `todo.md` 做任务规划，但约 **1/3 的操作都花在更新 todo 列表上**。转为专用 Planner Agent 后效率大幅提升。

### 两种上下文共享模式

| 模式 | 适用场景 | 做法 |
|------|----------|------|
| **轻量传递** | 离散任务 | 仅传指令 |
| **完整共享** | 复杂任务（需要共享文件） | 传递完整上下文 + 共享文件系统 |

子 Agent 通过 `submit results` 工具按 Schema 返回结果，用约束解码确保输出合规。

---

## 四、Context Offloading（上下文卸载）

### 分层操作空间

| 层级 | 内容 | Token 消耗 |
|------|------|-----------|
| **Function Calling 层** | < 20 个原子函数（Bash、文件系统、代码执行） | 低 |
| **Sandbox 层** | 通过 Bash 执行各种实用程序和 MCP CLI | 不占上下文 |

> 与 Claude Skills 理念一致：Skills 存储在文件系统中，通过 Bash 和文件操作按需发现和使用。

### 工具结果卸载

工具结果保存到文件系统，用 `glob` 和 `grep` 按需检索——不需要向量索引。

---

## 五、模型选择与缓存

### 任务级路由

| 模型 | 擅长领域 |
|------|----------|
| Claude | 编码 |
| Gemini | 多模态任务 |
| OpenAI | 数学和推理 |

### KV Cache 经济学

- 系统指令、旧工具结果等使用缓存减少成本和延迟
- 前沿提供商的缓存支持（如 Anthropic Prompt Caching）让商用模型在 Agent 场景下实际更便宜
- 分布式 KV Cache 在开源模型上很难实现

---

## 六、Bitter Lesson 与 Agent 工程

> 为当前模型能力添加结构来提升性能，但这些结构可能在模型更强时**成为瓶颈**。

**Manus 自 3 月发布以来已重构 5 次。**

Peak 的建议：
- 在**不同强度的模型**上跑评估
- 如果更强的模型没有带来性能提升，说明编排在限制 Agent
- 简单、无偏见的设计更容易适应模型进步

> "Add structures needed for the given level of compute and data available. Remove them later, because these shortcuts will bottleneck further improvement." — Hyung Won Chung (OpenAI)

---

## 七、核心启示

1. **工具结果的 Full/Compact 双重表示**——旧的用引用替代，新的保持完整
2. **多 Agent 的首要目标是隔离上下文，而非分工**——不要按人类角色划分
3. **todo.md 浪费 1/3 操作**——专用 Planner Agent 效率远高于文件式任务管理
4. **< 20 个原子函数 + Sandbox 层**——比绑定大量工具更高效
5. **文件系统 + grep/glob 就够了**——不一定需要向量索引
6. **任务级模型路由**——不同模型擅长不同任务
7. **KV Cache 是 Agent 经济性的关键**——缓存支持可能让商用模型反而更便宜
8. **不要让编排成为瓶颈**——跨模型强度测试，保持简单设计

---

## 参考资源

- [Manus 官方博客：Context Engineering](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)
- [Webinar 视频](https://youtu.be/6_BcCthVvb8?si=o8ovK6YNWOXtq7j7)
- [Peak 的演示文稿](https://docs.google.com/presentation/d/1Z-TFQpSpqtRqWcY-rBpf7D3vmI0rnMhbhbfv01duUrk/edit?usp=sharing)
- [E2B: How Manus uses E2B](https://e2b.dev/blog/how-manus-uses-e2b-to-provide-agents-with-virtual-computers)
- [Context Rot 研究](https://research.trychroma.com/context-rot)
- [Cognition: Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents)
- [The Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html)
- [Anthropic Prompt Caching](https://www.anthropic.com/news/prompt-caching)
