# Amazon Bedrock 多 Agent 协作：Supervisor 模式的全托管实现

> 原文链接：[Introducing multi-agent collaboration capability for Amazon Bedrock](https://aws.amazon.com/cn/blogs/aws/introducing-multi-agent-collaboration-capability-for-amazon-bedrock/)
>
> 日期：2024-12-01
>
> 作者：AWS（Amazon Web Services）
>
> 主题：agent, multi-agent, orchestration
>
> 类型：article

## 一、多 Agent 协作的核心问题

单个 Agent 在处理复杂多步骤任务时能力有限。当需要多个 Agent 协同工作时，开发者面临 **Agent 编排、会话管理、内存管理** 等技术挑战，开源方案需大量手动实现。

## 二、Amazon Bedrock 多 Agent 协作架构

采用 **Supervisor（监督者）模式**：

| 组件 | 角色 |
|------|------|
| **Supervisor Agent** | 分解请求、委派任务、汇总输出为最终响应 |
| **Sub-agents** | 在各自专业领域内执行具体任务 |

关键特点：Supervisor 负责协调，Sub-agents 专注于自己的领域，整个协作、通信和任务委派由 Bedrock 托管管理。

在内部基准测试中，多 Agent 协作相比单 Agent 系统在处理复杂多步骤任务时表现出 **显著改进**。

## 三、两种协作模式对比

| 模式 | 工作方式 | 适用场景 |
|------|----------|----------|
| **Supervisor 模式** | Supervisor 分析输入→分解问题→串行或并行调用子 Agent→处理响应→判断是否完成 | 复杂多步骤任务 |
| **Supervisor + Routing 模式** | 先尝试将简单请求**直接路由**到对应子 Agent；复杂/模糊输入自动回退到 Supervisor 模式 | 混合简单和复杂查询的场景 |

Routing 模式的优势：简单请求跳过完整编排流程，**提升响应效率**。

## 四、核心功能亮点

- **快速搭建**：几分钟内创建、部署和管理多 Agent 协作，无需复杂编码
- **可组合性（Composability）**：现有 Agent 可作为子 Agent 集成到更大的 Agent 系统中
- **高效通信**：Supervisor 使用一致接口与子 Agent 交互，支持**并行通信**
- **集成 Trace 调试控制台**：可视化分析多 Agent 交互过程
- **会话历史共享**：可选择是否将完整对话上下文传递给子 Agent（启用增强连贯性，禁用简化任务处理）

## 五、实战示例：社交媒体营销 Agent 系统

| Agent | 职责 |
|-------|------|
| **social-media-campaign-manager**（Supervisor） | 将子 Agent 输出组合为完整的营销方案 |
| **content-strategist**（Sub-agent） | 将商业目标转化为创意社交帖子（主题、内容类型、文案、标签） |
| **engagement-predictor**（Sub-agent） | 预测帖子表现和最佳发布时间（触达、互动率、最佳时段） |

每个子 Agent 可配置：
- 独立的 **指令（Instructions）**
- 独立的 **知识库（Knowledge Base）**
- 可选的 **Action Groups**、代码解释器、Guardrails

## 六、技术限制

- 子 Agent 本身可以启用协作功能，最多支持 **3 层层级 Agent 团队**
- 目前支持实时聊天（同步）场景

## 核心启示

- **Supervisor 模式是多 Agent 编排的主流范式**：分解-委派-汇总的三步流程简单有效
- **Routing + Supervisor 混合模式值得借鉴**：简单请求直接路由到专家 Agent，复杂请求才走完整编排，兼顾效率和能力
- **会话历史共享是个双刃剑**：增强连贯性但可能让简单子 Agent 困惑，需根据场景权衡
- **可组合性是关键设计原则**：已有 Agent 可无缝集成为子 Agent，支持渐进式构建复杂系统
- **与 Anthropic 多 Agent 系统对比**：Anthropic 侧重开源 SDK 层面的 orchestrator/handoff 模式，Bedrock 提供全托管的 Supervisor 模式，思路一致但抽象层次不同

## 参考资源

- [Amazon Bedrock Agents](https://aws.amazon.com/bedrock/agents/)
- [Amazon Bedrock Agent Samples（GitHub）](https://github.com/awslabs/amazon-bedrock-agent-samples/tree/main)
- [Multi-agent collaboration GA 公告](https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-announces-general-availability-of-multi-agent-collaboration/)
