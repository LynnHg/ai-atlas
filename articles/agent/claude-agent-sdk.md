# 用 Claude Agent SDK 构建 Agent

> 原文链接：[Building agents with the Claude Agent SDK](https://claude.com/blog/building-agents-with-the-claude-agent-sdk)
>
> 日期：2025-09-29
>
> 作者：Thariq Shihipar
>
> 主题：agent, claude-agent-sdk, tool-design, evaluation
>
> 类型：article

---

## 一、核心设计原则：给 Agent 一台计算机

Claude Code 的成功验证了一个关键洞察：与其给 Agent 专门的 API，不如给它程序员日常使用的工具——终端、文件系统、Bash 命令。

Claude Code SDK 更名为 **Claude Agent SDK**，因为同样的 Agent 基础设施不仅适用于编码，还能驱动深度研究、视频制作、笔记整理等各种通用 Agent。

> 文件夹和文件结构本身就是一种上下文工程。

---

## 二、Agent 循环

```
收集上下文 → 采取行动 → 验证工作 → 重复
```

---

## 三、收集上下文

| 方式 | 说明 | 特点 |
|------|------|------|
| **Agentic Search** | 用 grep、tail 等 Bash 命令按需加载文件 | 默认首选，准确、透明、易维护 |
| **Semantic Search** | 向量 embedding + 相似度查询 | 更快但不够准确，作为补充 |
| **Subagents** | 独立上下文并行搜索，只返回摘要 | 筛选大量信息时最佳 |
| **Compaction** | 接近上下文限制时自动摘要 | 长时间运行的 Agent 必备 |

子 Agent 的两大价值：
1. **并行化**：多个子 Agent 同时处理不同任务
2. **上下文管理**：独立上下文窗口，只返回相关信息

---

## 四、采取行动

| 方式 | 说明 | 案例 |
|------|------|------|
| **Tools（自定义工具）** | Agent 的主要操作，在上下文中突出 | `fetchInbox`、`searchEmails` |
| **Bash & Scripts** | 通用灵活能力 | 下载 PDF → 转文本 → 搜索内容 |
| **Code Generation** | 生成代码执行复杂操作 | 用 Python 创建 Excel/PPT/Word |
| **MCP** | 标准化外部服务集成 | 搜索 Slack、检查 Asana 任务 |

> 代码是精确、可组合、可复用的——考虑哪些任务适合用代码表达。Claude.AI 的文件创建功能完全依赖代码生成。

---

## 五、验证工作

| 方式 | 可靠性 | 说明 |
|------|--------|------|
| **Rules（规则验证）** | 高 | 明确规则 + 失败原因反馈。代码 Linting 是最佳范例 |
| **Visual Feedback（视觉反馈）** | 中 | 截图渲染结果，检查布局/样式/内容层次 |
| **LLM as Judge** | 低 | 另一个模型评判输出，延迟高但可用于语气等模糊标准 |

> 能自我验证的 Agent 根本性地更可靠——在错误复合前就捕获它们。

TypeScript > JavaScript 的一个原因：TypeScript 提供了更多层的规则反馈。

---

## 六、改进 Agent 的诊断思路

| 如果 Agent... | 可能的改进 |
|---------------|-----------|
| 误解任务 | 缺少关键信息。改进搜索 API 让信息更容易找到 |
| 反复在同一任务失败 | 在工具调用中添加规则来识别和修复失败 |
| 无法修复错误 | 给它更有用或更有创意的工具来换种方式解决 |
| 性能随功能增加而波动 | 基于用户使用模式构建代表性评估测试集 |

---

## 七、适用的 Agent 类型

- **金融 Agent**：理解投资组合、评估投资、访问外部 API、运行计算
- **个人助理 Agent**：订旅行、管日历、预约、准备简报、跨应用追踪上下文
- **客服 Agent**：处理高歧义请求、收集用户数据、对接 API、必要时转人工
- **深度研究 Agent**：在大文档集合中搜索、跨文件交叉引用、生成报告

---

## 八、核心启示

1. **"给 Agent 一台计算机"是核心设计原则**——终端 + 文件系统 + Bash 就够构建通用 Agent
2. **Agent 循环 = 收集上下文 + 采取行动 + 验证工作**——这是所有 Agent 的通用模式
3. **文件系统搜索优先于语义搜索**——更准确、更透明、更易维护
4. **代码生成是被低估的 Agent 能力**——精确、可组合、可复用
5. **自验证是可靠性的关键**——规则 > 视觉反馈 > LLM 评判（可靠性递减）
6. **改进 Agent 要从失败案例出发**——站在 Agent 的角度问"它有足够的工具吗？"

---

## 参考资源

- [Claude Agent SDK 文档](https://docs.claude.com/en/api/agent-sdk/overview)
- [Claude Agent SDK 迁移指南](https://docs.claude.com/en/docs/claude-code/sdk/migration-guide)
- [自定义工具文档](https://docs.claude.com/en/api/agent-sdk/custom-tools)
- [Subagents 文档](https://docs.claude.com/en/api/agent-sdk/subagents)
- [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval)
