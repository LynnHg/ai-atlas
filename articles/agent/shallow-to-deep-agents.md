# Agent 2.0：从浅层循环到深层 Agent

> 原文链接：[Agents 2.0: From Shallow Loops to Deep Agents](https://www.philschmid.de/agents-2.0-deep-agents)
>
> 日期：2025-10-12
>
> 作者：Philipp Schmid
>
> 主题：agent, deep-agent, multi-agent, context-engineering, planning
>
> 类型：article

---

## 一、Agent 1.0（浅层 Agent）的局限

浅层 Agent = while 循环中的 LLM + 工具调用。整个"大脑"就是上下文窗口。

```
用户提示 → LLM 推理 → 工具调用 → 观察结果 → 生成回答 → 重复
```

适合 **5-15 步**的事务性任务，但在 500 步的复杂任务上失败。

| 问题 | 说明 |
|------|------|
| **上下文溢出** | 工具输出填满窗口，指令被挤出 |
| **目标丢失** | 在中间步骤的噪声中忘记原始目标 |
| **无恢复机制** | 进入死胡同后无法回退换策略 |

---

## 二、Agent 2.0（深层 Agent）的四大支柱

### 支柱 1：显式规划

| 浅层 Agent | 深层 Agent |
|------------|------------|
| 隐式规划（CoT） | 用工具创建和维护**显式计划**（如 To-Do 列表） |

- 每步之间回顾和更新计划
- 标记状态：pending → in_progress → completed
- 失败时更新计划以适应，而非盲目重试

### 支柱 2：层级委派（Sub-Agents）

Orchestrator → 子 Agent 模式：
- 编排者委派任务给专门的子 Agent（研究员、编码者、写手）
- 每个子 Agent 有**干净的上下文**
- 子 Agent 完成完整的工具调用循环
- 只将**综合后的答案**返回编排者

### 支柱 3：持久记忆

从"记住一切"到"知道去哪里找信息"：
- 使用文件系统或向量数据库作为外部存储
- Agent 将中间结果（代码、草稿、数据）写入外部
- 后续 Agent 通过文件路径或查询按需检索
- 防止上下文窗口溢出

### 支柱 4：极致上下文工程

> 更强的模型不需要更少的提示，而是需要**更好的上下文**。

深层 Agent 的详细指令定义：
- 何时停下来先规划再行动
- 何时派生子 Agent vs 自己处理
- 工具定义和使用示例
- 文件命名和目录结构标准
- 人机协作的严格格式

---

## 三、深层 Agent 的完整流程

```
用户请求（复杂任务）
  → 创建显式计划（To-Do 列表）
    → 选择第一个任务
      → 判断：自己做还是委派子 Agent？
        → 子 Agent 在独立上下文中执行
          → 写中间结果到文件系统
          → 返回综合答案
      → 更新计划（标记完成/添加新步骤）
    → 选择下一个任务 → 重复...
  → 所有任务完成 → 最终输出
```

---

## 四、核心启示

1. **Agent 1.0 → 2.0 不是加更多工具，而是架构升级**——从反应式循环到主动式架构
2. **四大支柱缺一不可**：显式规划 + 层级委派 + 持久记忆 + 极致上下文工程
3. **从"记住一切"到"知道去哪里找"**——持久记忆的范式转变
4. **控制上下文 = 控制复杂度**——解锁小时级/天级任务的关键
5. **更强的模型需要更好（而非更少）的上下文**
6. **显式计划是防止目标丢失和盲目重试的核心机制**
7. **浅层 Agent 适合 5-15 步，深层 Agent 面向 500 步**

---

## 参考资源

- [LangChain: Deep Agents](https://blog.langchain.com/deep-agents/)
- [Deep Agents 开源项目](https://github.com/langchain-ai/deepagents)
- [Philipp Schmid: Agentic Patterns](https://www.philschmid.de/agentic-pattern)
- [Philipp Schmid: Memory in Agents](https://www.philschmid.de/memory-in-agents)
- [Philipp Schmid: The Rise of Sub-Agents](https://www.philschmid.de/the-rise-of-subagents)
