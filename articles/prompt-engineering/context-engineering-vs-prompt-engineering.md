# 上下文工程 vs. 提示词工程：从 2D 到 3D 的升维

> 原文链接：[Context engineering vs. prompt engineering](https://port.io/blog/context-engineering-vs-prompt-engineering)
>
> 日期：2025-11-24
>
> 作者：Port.io
>
> 主题：prompt-engineering, context-engineering, agent
>
> 类型：article

## 核心概念对比

文章用一个**立方体类比**来解释两者的关系——提示词工程是看到立方体的一个面（2D），上下文工程是看到完整的立方体（3D）。

| 维度 | 提示词工程 (Prompt Engineering) | 上下文工程 (Context Engineering) |
|------|------|------|
| **类比** | 立方体的一个面（2D） | 完整的立方体（3D） |
| **定义** | 精心设计指令，告诉 AI 去哪里找数据、如何使用、如何返回结果 | 为 AI 提供历史数据、记忆、实体关系等完整上下文信息 |
| **产出特点** | 一致的、可重复的动作，可控制输出格式 | Agent 能自主理解请求、决定响应去向、规划下一步行动 |
| **适用场景** | 需要**确定性 AI**，解决特定问题，提供可靠可重复的结果 | 需要 Agent 在复杂 SDLC 中深度执行多步异步工作流 |
| **扩展性** | 依赖手动调优指令，扩展性有限 | 自动化信息访问，减少重写提示词的需要 |

## 两者是互补放大关系

上下文工程**包含并提升**了提示词工程，而非取代。即使有了强大的上下文管道，精心设计的提示词仍然可以：

- **设置护栏**（guardrails）——约束 Agent 行为边界
- **引导 Agent 到正确结果**——指定数据使用方式和行动规则
- **减少幻觉发生**——虽然提示词优化能降低幻觉概率，但上下文工程**更一致地产生准确结果**

| 方法 | 对幻觉的效果 |
|------|------|
| 仅提示词工程 | 可以降低幻觉概率，但不够稳定 |
| 上下文工程 | 更一致地产生准确结果，因为它给 Agent 足够可靠的数据 |

## MCP 与上下文湖

MCP 服务器是上下文工程的关键基础设施：

- MCP 服务器为 AI Agent 提供对**上下文湖（Context Lake）** 的访问
- 上下文湖包含 Agent 正确执行任务所需的所有数据
- 有了正确的上下文，Agent 可以像人类员工一样在平台中深度行动
- Agent 能独立执行从自助服务操作到推送代码上线的多步异步工作流

## 实战案例：自愈式事件处理

以支付服务延迟告警为例，展示两者如何协同工作：

| 步骤 | 工程类型 | 具体操作 |
|------|---------|---------|
| 拉取领域数据 | 上下文工程 | Agent 通过 MCP 访问上下文湖，获取支付服务的领域集成信息 |
| 分析根因 | 上下文工程 | 自主根因分析 Agent 拉取信息，交给编码 Agent 审查 |
| 设置行为约束 | 提示词工程 | 平台嵌入护栏提示词 |

护栏提示词示例：

- `"仅使用提供的上下文和引用证据"`
- `"如果置信度 < 0.6，请求人工审核而非尝试修改"`
- `"遵循此风险策略：中等风险 + 需要审批"`

最终结果：Agent 能自主解决事件，或在需要时识别并请求人工审核。

## 对开发者角色的影响

文章指出，随着 AI Agent 深度嵌入工作流，开发者的角色将从**"主要编写代码"** 转变为**"主要编写提示词来让 AI Agent 编写代码"**。结合提示词工程和上下文工程是持续优化团队敏捷性的关键。

## 核心启示

1. **上下文工程不会让提示词工程过时**——两者是 2D 到 3D 的升维关系，应结合使用
2. **上下文的质量直接决定 Agent 输出的质量**——好的上下文能让简单的提示词也产生出色结果
3. **MCP + 上下文湖是上下文工程的核心基础设施**——让 Agent 能自主访问结构化的领域知识
4. **提示词工程擅长设定护栏和约束**——即使上下文工程已就位，提示词仍然承担着关键的行为控制角色
5. **确定性场景用提示词工程，复杂场景两者结合**——根据任务复杂度选择合适的策略

## 参考资源

- [Port.io 术语表 - Prompt Engineering](https://www.port.io/glossary/prompt-engineering)
- [Port.io 术语表 - Context Engineering](https://www.port.io/glossary/context-engineering)
- [Port.io 术语表 - Context Lake](https://www.port.io/glossary/context-lake)
- [Port.io 术语表 - Deterministic AI](https://www.port.io/glossary/deterministic-ai)
- [Port MCP Server 集成](https://www.port.io/blog/integrate-software-catalog-every-workflow-port-mcp-server)
- [Agentic Chaos 风险](https://www.port.io/blog/risks-agentic-chaos)
