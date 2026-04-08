# OpenAI 新 Agent 平台：Responses API、Web Search、Computer Use 与 Agents SDK

> 原文链接：[⚡️The new OpenAI Agents Platform](https://www.latent.space/p/openai-agents-platform)
>
> 日期：2025-03-11
>
> 作者：Nikunj Handa, Romain Huet（OpenAI）× Latent Space
>
> 主题：agent, openai, multi-agent, sdk
>
> 类型：article

## 1. Responses API：Chat Completions 的进化

OpenAI 推出 Responses API，作为 Chat Completions API 和 Assistants API 的**统一替代**：

| 对比维度 | Chat Completions | Assistants API | Responses API |
|----------|-----------------|----------------|---------------|
| 状态管理 | 无状态 | 有状态（复杂对象模型） | 默认有状态（免费存储 30 天），可关闭 |
| 内置工具 | 不支持 | 支持（即将淘汰） | 支持（Web Search、Computer Use、File Search） |
| 复杂度 | 低 | 高（6 个对象） | 低（单次 API 请求） |
| 定位 | 单轮对话 | 长期会话 | **agentic 工作流**（多轮、多工具组合） |

关键信息：

- Chat Completions **不会下线**，继续支持新模型
- Assistants API 计划**2025 上半年 sunset**
- Responses API 是新用户的推荐起点
- 免费存储状态 30 天，内置 Dashboard 可视化调试

## 2. 三大内置工具

| 工具 | 功能 | 定价 | 技术细节 |
|------|------|------|----------|
| **Web Search** | ChatGPT Search 能力开放给 API | $25-30/1000 queries | 基于 GPT-4o 微调模型，支持**内联引用**（精确到段落级深链接） |
| **Computer Use** | Operator 的底层能力（CUA 模型） | $3/1M input, $12/1M output | 输入截图 → 输出操作指令；OSWorld **38.1%**，WebVoyager **87%** |
| **File Search** | 托管 RAG 服务 | $2.50/1000 queries + $0.10/GB/day | 新增**元数据过滤**，自动处理解析/分块/嵌入 |

核心亮点：这三个工具可以在**单次 Responses API 调用**中组合使用。例如：File Search 获取用户偏好 → Web Search 搜索匹配内容 → 结构化输出 JSON。

### Web Search

- 在 Responses API 中以**工具**形式提供；在 Chat Completions 中以**独立模型**（`gpt-4o-search-preview`）提供
- 核心优势：内联引用——不仅返回页面链接，还深链到回答所在的**具体段落**
- Simple QA 准确率从 GPT-4o 的 **38%** 跃升到 Search 版本的 **90%**
- 使用合成数据 + o 系列模型蒸馏来提升事实性和引用准确性

### Computer Use

- 与 Operator 产品使用同一底层模型 `computer-use-preview`
- 工作方式：发送截图 → 模型返回操作指令（点击/滚动/输入）
- 团队自评处于"GPT-1 / GPT-2 阶段"，仍是非常早期
- 演进方向：最终会合并进主线模型（类似 Vision 从 preview 到内置的路径）

### File Search

- 全托管 RAG：上传文件 → 自动解析、分块、嵌入、构建向量存储
- 新增元数据过滤（在向量存储超过 5,000-10,000 条记录后至关重要）
- 创意用法：将用户偏好/记忆存入向量存储，结合 Web Search 做个性化推荐

## 3. Agents SDK：从 Swarm 到正式框架

Swarm 原本是实验性项目，因社区热烈反响升级为官方支持的 Agents SDK，包含 **4 个核心组件**：

| 组件 | 功能 |
|------|------|
| **Agents** | 可配置的 LLM + 指令 + 工具 |
| **Handoffs** | Agent 间智能控制转移 |
| **Guardrails** | 输入/输出安全校验（与生成**并行**执行，可阻断） |
| **Tracing** | 执行链路可视化，集成 OpenAI Dashboard |

支持的 Agent 模式：Workflows、Handoffs、Agents-as-Tools、LLM-as-a-Judge、Parallelization、Guardrails。

**开放性设计**：

- 兼容任何支持 ChatCompletions 格式的 API 提供商
- 支持多种 tracing 提供商（不绑定 OpenAI Dashboard）
- 加入 TypeScript 类型支持

## 4. 未来愿景：Traces → Evals → RFT 闭环

OpenAI 规划的完整闭环：**Agent 执行 → Traces 记录 → 生成 Evals → 强化微调（RFT）**，让开发者能持续优化自己的 Agent 表现。

## 核心启示

- **API 统一趋势**：Responses API 是 OpenAI 押注 Agent 范式的核心基础设施，"一个 API 调用搞定一切"
- **Computer Use 仍是早期**：团队自比"GPT-1/GPT-2 阶段"，但已 SOTA，后续会合并进主线模型
- **托管 RAG vs 自建 RAG**：OpenAI 建议"先用我们的，不够再自建"——对简单场景够用，深度定制仍需自建
- **Web Search 的差异化**：段落级精确引用是核心竞争力，且可与 function calling + structured output 组合
- **Handoff 模式成为主流**：从 Swarm 的实验验证到 SDK 的正式支持，多 Agent 编排进入生产就绪阶段
- **可观测性是一等公民**：Tracing 内置于 SDK 和 Dashboard，而非事后补充

## 参考资源

- [OpenAI Agents SDK 文档](https://platform.openai.com/docs/guides/agents)
- [Agent Patterns 示例](https://github.com/openai/openai-agents-python/tree/main/examples/agent_patterns)
- [Swarm 原始仓库](https://github.com/openai/swarm/)
- [OpenAI DevDay 2024 回顾](https://www.latent.space/p/devday-2024)
- [Exa AI 播客（Web Search 对比）](https://www.latent.space/p/exa)
