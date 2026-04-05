# Skills vs MCP：为什么我把所有 MCP 都迁移到了 Skills

> 原文链接：[Skills vs Dynamic MCP Loadouts](https://lucumr.pocoo.org/2025/12/13/skills-vs-mcp/)
>
> 日期：2025-12-13
>
> 作者：Armin Ronacher（Flask 作者）
>
> 主题：mcp, skills, agent, context-engineering, tool-design
>
> 类型：article

---

## 一、核心论点

Armin 已将所有 MCP（Sentry、Playwright 等）迁移到 Skills。原因不是 MCP 不好，而是 Skills 的工程成本更低、实际效果更好。

---

## 二、工具加载的底层机制

### 传统 MCP

- 工具定义在对话开始时加载到系统提示
- 中途修改 → 失去推理轨迹 + 失去 Cache → 全价 Token 重新计算

### 延迟加载（Deferred Tool Loading）

- 预声明但初始不注入，后续按需加载
- 工具定义必须静态定义，整个对话不变
- 通过正则搜索发现工具

---

## 三、Skills 的优势

| 维度 | MCP | Skills |
|------|-----|--------|
| 上下文成本 | Sentry MCP ~**8k Token** | 只加载简短摘要（几十 Token） |
| 工具机制 | 引入新工具定义 | **不引入新工具**，教 Agent 更好使用已有工具（Bash 等） |
| 工程复杂度 | 需要 API 层工程 | 零额外工程，只是 Markdown 文件 |
| 与 RL 的配合 | 新工具需要 Agent 重新学习 | 复用 Agent 通过 RL 学会的工具调用能力 |
| API 稳定性 | Server 经常变更描述 | 开发者自己控制，稳定 |

> 关键洞察：Skills 不加载新工具定义——Agent 始终使用 Bash 等基础工具。Claude 通过 RL 获得的工具调用能力被完全复用。

---

## 四、MCP 通过 CLI 暴露的尝试

使用 mcporter 将 MCP 暴露为 CLI 命令：

```bash
npx mcporter call 'linear.create_comment(issueId: "ENG-123", body: "Looks good!")'
```

问题：
- Agent 不知道可用工具，需要额外教它
- 最终还是要手动维护 Skill 摘要
- MCP Server 频繁改描述，摘要很快过时

> MCP 描述的尴尬折中：**太长无法预加载，太短无法指导使用**。

---

## 五、最省力路径：让 Agent 自己写工具

> "让 Agent 自己写工具作为 Skill。Skill 可能有 Bug，但因为 Agent 自己维护它，反而运转得更好。"

Sentry 案例：
- MCP 预加载消耗 8k Token，mcporter 也用不好
- 改为让 Claude 维护 Sentry Skill
- Agent 需要时自己调整工具——比依赖外部 MCP Server 更灵活

---

## 六、MCP 的当前问题

| 问题 | 说明 |
|------|------|
| API 不稳定 | 工具描述随意变更，破坏外部引用 |
| 描述过于精简 | 为省 Token 压缩到最低，Skill 模式需要更详细说明 |
| 缺少内置文档 | 没有 Skill 式的手册和使用指南 |
| 认证复杂 | Linear、Sentry 都有认证问题 |

> MCP 需要：协议稳定性 + Skill 式摘要 + 内置工具手册。

---

## 七、核心启示

1. **Skills 不引入新工具定义**——只教 Agent 更好地使用已有工具，复用 RL 能力
2. **MCP 描述的尴尬折中**——太长无法预加载，太短无法指导使用
3. **让 Agent 自己维护工具**——比手动维护 MCP 集成更灵活
4. **MCP API 不稳定是根本问题**——频繁变更破坏外部引用
5. **8k Token 预加载成本很可观**——一个 MCP Server 就占用大量上下文
6. **修改工具定义导致 Cache 失效**——全价 Token 重算，代价巨大
7. **未来 MCP 可能融合 Skill 模式**——需要协议级摘要和文档支持

---

## 参考资源

- [Armin Ronacher: Tools 实验](https://lucumr.pocoo.org/2025/7/3/tools/)
- [mcporter 项目](https://github.com/steipete/mcporter)
- [Anthropic: Advanced Tool Use](https://www.anthropic.com/engineering/advanced-tool-use)
- [Simon Willison: Claude Skills > MCP](https://simonwillison.net/2025/Oct/16/claude-skills/)
