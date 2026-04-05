# Claude 高级工具使用：搜索、编程式调用与示例

> 原文链接：[Introducing advanced tool use on the Claude Developer Platform](https://www.anthropic.com/engineering/advanced-tool-use)
>
> 日期：2025-11-24
>
> 作者：Bin Wu
>
> 主题：agent, tool-use, mcp, programmatic-tool-calling, context-engineering
>
> 类型：article

---

## 一、问题背景：工具越多，Agent 越难用

一个典型的多 MCP Server 场景：

| MCP Server | 工具数 | Token 消耗 |
|------------|--------|-----------|
| GitHub | 35 | ~26K |
| Slack | 11 | ~21K |
| Jira | - | ~17K |
| Sentry | 5 | ~3K |
| Grafana | 5 | ~3K |
| Splunk | 2 | ~2K |

仅工具定义就消耗 **55K-134K Token**，对话还没开始上下文就快满了。而且最常见的失败是**选错工具**和**参数错误**，尤其是名称相似的工具（如 `notification-send-user` vs `notification-send-channel`）。

三大瓶颈：
1. 工具定义的**上下文膨胀**
2. 中间结果的**上下文污染**
3. 复杂参数的**调用错误**

---

## 二、Tool Search Tool（工具搜索工具）

### 核心思路

不预加载所有工具定义，Agent 按需搜索和发现工具。

### 对比数据

| 对比项 | 传统方式 | Tool Search Tool |
|--------|----------|------------------|
| 初始加载 | 所有工具（~72K Token） | 仅搜索工具本身（~500 Token） |
| 使用时加载 | - | 3-5 个相关工具（~3K Token） |
| 总消耗 | ~77K Token | ~8.7K Token |
| Token 节省 | - | **85%** |
| 可用上下文 | 122,800 Token | **191,300 Token** |

准确率提升：
- Opus 4：49% → **74%**
- Opus 4.5：79.5% → **88.1%**

### 工作原理

1. 给工具标记 `defer_loading: true`，延迟加载
2. 保留 3-5 个高频工具 `defer_loading: false` 始终加载
3. Claude 需要某功能时，调用 Tool Search Tool 搜索
4. 匹配的工具定义才被展开到上下文中

不影响 Prompt 缓存——延迟工具完全不在初始 Prompt 中。

### 实现示例

```json
{
  "tools": [
    {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},
    {
      "name": "github.createPullRequest",
      "description": "Create a pull request",
      "input_schema": {},
      "defer_loading": true
    }
  ]
}
```

对 MCP Server 可以整体延迟加载，只保留最常用的工具：

```json
{
  "type": "mcp_toolset",
  "mcp_server_name": "google-drive",
  "default_config": {"defer_loading": true},
  "configs": {
    "search_files": {"defer_loading": false}
  }
}
```

### 适用条件

- 工具定义 > 10K Token
- 10+ 个工具
- 多 MCP Server 场景
- 工具选择准确率有问题

---

## 三、Programmatic Tool Calling（编程式工具调用）

### 核心思路

让 Claude 写 Python 代码编排工具调用，中间结果在沙箱代码环境中处理，只把最终结果返回上下文。

### 传统方式的问题

- 每次工具调用需要一次完整模型推理
- 所有中间结果（哪怕无用）都堆积在上下文中
- Claude 需要用自然语言"肉眼"解析和对比数据

### 对比案例：查找超预算的团队成员

| 对比项 | 传统方式 | Programmatic Tool Calling |
|--------|----------|--------------------------|
| 工具调用 | 20+ 次，每次一个推理 pass | 1 个代码块，批量并行 |
| 进入上下文的数据 | 2000+ 行费用明细（200KB） | 仅最终结果（~1KB） |
| 推理次数 | 20+ 次 | 1 次 |

Claude 生成的编排代码：

```python
team = await get_team_members("engineering")

levels = list(set(m["level"] for m in team))
budget_results = await asyncio.gather(*[
    get_budget_by_level(level) for level in levels
])
budgets = {level: budget for level, budget in zip(levels, budget_results)}

expenses = await asyncio.gather(*[
    get_expenses(m["id"], "Q3") for m in team
])

exceeded = []
for member, exp in zip(team, expenses):
    budget = budgets[member["level"]]
    total = sum(e["amount"] for e in exp)
    if total > budget["travel_limit"]:
        exceeded.append({
            "name": member["name"],
            "spent": total,
            "limit": budget["travel_limit"]
        })

print(json.dumps(exceeded))
```

### 关键数据

- Token 消耗：43,588 → 27,297（降低 **37%**）
- 内部知识检索准确率：25.6% → **28.5%**
- GIA 基准：46.5% → **51.2%**

### 工作原理

1. 给工具添加 `allowed_callers: ["code_execution_20250825"]`
2. Claude 生成 Python 编排代码（支持 asyncio 并行）
3. 代码中的工具调用在沙箱环境执行，结果传给代码而非 Claude 上下文
4. 只有代码最终的 `print` 输出进入上下文

### 适用条件

- 大数据集处理（只需聚合/摘要）
- 3+ 步依赖工作流
- 需要过滤/排序/转换中间结果
- 中间数据不应影响 Claude 推理
- 可并行操作（如检查 50 个端点）

---

## 四、Tool Use Examples（工具使用示例）

### 核心思路

JSON Schema 只能定义结构合法性，无法表达**使用模式**。通过 `input_examples` 直接在工具定义中提供示例。

### Schema 无法回答的问题

- `due_date` 用什么格式？`"2024-11-06"` 还是 `"Nov 6, 2024"`？
- `reporter.id` 是 UUID 还是 `"USR-12345"`？
- 什么时候该填 `escalation` 嵌套对象？
- `priority` 和 `escalation.sla_hours` 之间什么关系？

### 关键数据

复杂参数处理准确率：72% → **90%**

### 示例设计原则

- 使用**真实数据**（真实城市名、合理价格，不要 `"string"` 占位符）
- 展示**多样性**：最小参数 / 部分参数 / 完整参数三种模式
- 保持**精简**：每个工具 1-5 个示例
- 聚焦**歧义点**：只在 Schema 无法表达正确用法的地方添加

### 适用条件

- 复杂嵌套结构
- 大量可选参数且填写模式重要
- API 有领域特定惯例（Schema 无法捕获）
- 相似工具需要区分使用场景

---

## 五、三大功能的组合策略

按瓶颈选择，逐步叠加：

| 你的瓶颈 | 首选方案 |
|----------|----------|
| 工具定义占用太多上下文 | Tool Search Tool |
| 大量中间结果污染上下文 | Programmatic Tool Calling |
| 参数错误、调用格式不对 | Tool Use Examples |

三者互补形成完整闭环：

```
Tool Search Tool → 找到正确的工具
Tool Use Examples → 确保正确调用
Programmatic Tool Calling → 确保高效执行
```

---

## 六、核心启示

1. **工具定义是 Agent 系统中最被低估的 Token 消耗源**——134K Token 的工具定义比很多对话还长
2. **按需发现 > 全量预加载**——85% 的 Token 节省 + 更高的选择准确率
3. **代码编排 > 自然语言编排**——让模型用 Python 而非自然语言处理数据流，更精确、更可控
4. **中间结果不该进上下文**——Agent 只需看最终结果，不需要看原始数据
5. **示例比 Schema 更有效地表达使用模式**——复杂参数准确率从 72% 到 90%
6. **先解决最大瓶颈，再逐步叠加**——不要一次性使用全部功能

---

## 参考资源

- [Tool Search Tool 文档](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool)
- [Tool Search Tool Cookbook（Embedding 方案）](https://github.com/anthropics/claude-cookbooks/blob/main/tool_use/tool_search_with_embeddings.ipynb)
- [Programmatic Tool Calling 文档](https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling)
- [Programmatic Tool Calling Cookbook](https://github.com/anthropics/claude-cookbooks/blob/main/tool_use/programmatic_tool_calling_ptc.ipynb)
- [Tool Use Examples 文档](https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use#providing-tool-use-examples)
- [Building effective agents](https://www.anthropic.com/research/building-effective-agents)
- [Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)
- [GIA Benchmark](https://arxiv.org/abs/2311.12983)
