# 为 Agent 编写高效工具——用 Agent 来优化

> 原文链接：[Writing effective tools for agents — with agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
>
> 日期：2025-09-11
>
> 作者：Ken Aizawa
>
> 主题：agent, tool-design, mcp, evaluation, prompt-engineering
>
> 类型：article

---

## 一、核心认知：工具 ≠ API

传统软件是确定性系统之间的契约：`getWeather("NYC")` 每次行为完全一致。

工具是**确定性系统与非确定性 Agent 之间的契约**：用户问"今天该带伞吗？"，Agent 可能调用天气工具、用已有知识回答、或先追问地点。

> 不能像给开发者写 API 一样给 Agent 写工具——需要**为 Agent 设计**。最"人体工学"的工具对人类来说也出奇直观。

---

## 二、工具开发方法论

### 2.1 快速原型

- 用 Claude Code 一次性生成工具原型
- 提供 `llms.txt` 格式的文档帮助 Claude 理解依赖的 SDK/API
- 包装为本地 MCP Server 或 DXT，在 Claude Code / Claude Desktop 中测试
- 自己先测试，收集用户反馈，建立直觉

### 2.2 构建评估

评估任务的设计至关重要：

| 强评估任务 | 弱评估任务 |
|------------|------------|
| "安排下周和 Jane 讨论 Acme 项目，附上规划会笔记并预定会议室" | "安排一个和 jane@acme.corp 的会议" |
| "客户 9182 报告被扣三次费。找相关日志，判断是否影响其他客户" | "搜索 payment logs 的 purchase_complete" |
| "客户 Sarah Chen 提交取消请求。准备留客方案：离开原因、最佳留客策略、风险因素" | "找客户 45892 的取消请求" |

强任务特征：基于真实场景、需要多步工具调用、不预设策略。

运行评估的建议：
- 使用简单的 while 循环包装 LLM API + 工具调用
- 让 Agent 输出推理和反馈块（触发 CoT），不只是结构化响应
- 收集指标：准确率、运行时间、工具调用数、Token 消耗、错误

### 2.3 与 Agent 协作迭代

将评估 transcript 丢给 Claude Code，让它：
- 分析失败原因
- 发现矛盾的工具描述
- 重构工具实现，保持一致性

> 文章中的大部分建议来自反复用 Claude Code 优化内部工具的过程。使用留出测试集防止过拟合。

---

## 三、五大工具设计原则

### 3.1 选择正确的工具（少即是多）

不要简单包装 API 端点。Agent 的上下文有限，不能暴力遍历。

| 不要这样做 | 应该这样做 |
|------------|------------|
| `list_users` + `list_events` + `create_event` | `schedule_event`（一步找空闲并创建） |
| `read_logs` | `search_logs`（只返回相关日志 + 上下文） |
| `get_customer_by_id` + `list_transactions` + `list_notes` | `get_customer_context`（一步编译全部信息） |

核心思路：
- 工具可以在内部整合多个操作，减少调用次数和上下文消耗
- 每个工具要有清晰、独特的用途
- 过多或重叠的工具会分散 Agent 注意力
- 仅针对高影响力工作流构建工具

### 3.2 工具命名空间化

当 Agent 接入数十个 MCP Server 时，命名空间帮助区分：
- 按服务：`asana_search`、`jira_search`
- 按资源：`asana_projects_search`、`asana_users_search`

> 前缀命名 vs 后缀命名对评估结果有非平凡的影响，需根据自己的评估选择。

### 3.3 返回有意义的上下文

优先返回高信号信息，而非原始技术标识符：

| 避免 | 推荐 |
|------|------|
| `uuid` | `name` |
| `256px_image_url` | `image_url` |
| `mime_type` | `file_type` |

将 UUID 解析为语义化名称可**显著降低幻觉**。

提供 `response_format` 参数控制返回详细度：

```
enum ResponseFormat {
   DETAILED = "detailed",   // 完整信息含 ID（206 tokens）
   CONCISE = "concise"      // 仅核心内容（72 tokens，约 1/3）
}
```

`"detailed"` 用于需要 ID 进行后续工具调用的场景；`"concise"` 用于只需内容的场景。

> 工具响应的格式（XML、JSON、Markdown）也会影响性能，没有万能方案，需要评估。

### 3.4 优化 Token 效率

- 实现**分页、范围选择、过滤、截断**，设置合理默认值
- Claude Code 默认限制工具响应为 **25,000 Token**
- 截断响应时附带引导信息，引导 Agent 使用过滤或分页
- 错误响应要**具体可操作**：

| 糟糕的错误响应 | 好的错误响应 |
|----------------|-------------|
| `Error 400: Bad Request` | "日期格式错误，请使用 YYYY-MM-DD 格式。示例：2025-09-11" |

### 3.5 Prompt 工程化工具描述

- 像给团队新人介绍工具一样写描述
- 把隐含知识显式化：查询格式、术语定义、资源间关系
- 参数命名要明确：用 `user_id` 而非 `user`

> SWE-bench 案例：Claude Sonnet 3.5 仅通过精确微调工具描述就达到了 SOTA。Web Search 工具案例：发现 Claude 会无谓地在查询后追加 "2025"，改进描述后消除了这一行为。

---

## 四、核心启示

1. **工具是为非确定性系统设计的新型软件**——不能像写 API 一样写工具
2. **少而精 > 多而泛**——几个精心设计的高影响力工具好过大量 API 包装
3. **工具可以整合多步操作**——减少 Agent 的调用次数和上下文消耗
4. **返回高信号信息，避免原始标识符**——语义化名称显著降低幻觉
5. **`response_format` 参数是利器**——让 Agent 自己控制返回详细度
6. **错误响应要引导 Agent 自我纠正**——具体、可操作 > 晦涩错误码
7. **评估驱动开发**——让 Claude 分析 transcript 并改进工具，形成迭代循环
8. **工具描述的微小改进可以产生巨大影响**——这是最有效的优化手段之一

---

## 参考资源

- [工具评估 Cookbook](https://platform.claude.com/cookbook/tool-evaluation-tool-evaluation)
- [MCP SDK 文档](https://modelcontextprotocol.io/docs/sdk)
- [Anthropic 工具使用指南](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use)
- [MCP 工具注解规范](https://modelcontextprotocol.io/specification/2025-06-18/server/tools)
- [Anthropic llms.txt](https://docs.anthropic.com/llms.txt)
- [SWE-bench 工具优化案例](https://www.anthropic.com/engineering/swe-bench-sonnet)
