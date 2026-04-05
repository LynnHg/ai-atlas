# 长时运行 Agent 的有效编排：跨上下文窗口的增量进展

> 原文链接：[Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
>
> 日期：2025-11-26
>
> 作者：Justin Young
>
> 主题：agent, long-horizon, context-engineering, evaluation, claude-agent-sdk
>
> 类型：article

---

## 一、核心问题：跨上下文窗口的记忆断裂

长时运行 Agent 必须在离散的会话中工作，每个新会话开始时对之前发生的一切毫无记忆。即使有 Compaction，前沿模型仍会在复杂项目中失败。

### 两大失败模式

| 失败模式 | 表现 |
|----------|------|
| **一次性尝试过多** | Agent 试图一口气完成所有功能，上下文耗尽时功能半实现、无文档。下一个 Agent 花大量时间猜测 |
| **过早宣布完成** | 看到已有一些功能，就认为项目已完成 |

---

## 二、解决方案：双 Agent 结构

| Agent | 职责 | 运行时机 |
|-------|------|----------|
| **Initializer Agent** | 搭建初始环境（init.sh、进度文件、功能清单、初始 git commit） | 仅第一次运行 |
| **Coding Agent** | 每次会话做增量进展，结束时留下干净状态和结构化更新 | 每次后续运行 |

> 核心洞察：让 Agent 在启动新上下文窗口时能**快速理解工作状态**——通过 `claude-progress.txt` + git 历史实现。灵感来自人类工程师的日常实践。

---

## 三、环境管理的关键组件

### 3.1 功能清单（Feature List）

Initializer Agent 将高级提示展开为详细的功能列表（如 200+ 项），每项初始标记 `"passes": false`：

```json
{
  "category": "functional",
  "description": "New chat button creates a fresh conversation",
  "steps": [
    "Navigate to main interface",
    "Click the 'New Chat' button",
    "Verify a new conversation is created",
    "Check that chat area shows welcome state"
  ],
  "passes": false
}
```

关键规则：
- Coding Agent 只能修改 `passes` 字段，**不能删除或编辑测试内容**
- 使用 JSON 而非 Markdown——模型更不容易不当修改 JSON 文件

### 3.2 增量进展

每次 Coding Agent 只处理**一个功能**。每次会话结束必须：
- Git commit + 描述性提交信息
- 更新 `claude-progress.txt` 进度文件
- 代码保持可合并状态：无重大 Bug、有序、有文档

下一个 Agent 可以用 git revert 回退坏代码，恢复工作状态。

### 3.3 测试验证

常见问题：Agent 标记功能为完成但没有真正端到端测试。

解决方案：明确提示使用浏览器自动化工具（如 Puppeteer MCP），**像人类用户一样测试**。

> 提供测试工具后性能大幅提升——Agent 能发现仅看代码看不出的 Bug。

局限：Claude 视觉能力有限，无法看到浏览器原生 alert 弹窗等。

---

## 四、每次会话的启动流程

```
1. pwd → 确认工作目录
2. 读 git log + claude-progress.txt → 了解最近工作
3. 读 feature_list.json → 选择最高优先级的未完成功能
4. 运行 init.sh → 启动开发服务器
5. 基础端到端测试 → 确认应用没被破坏
6. 开始实现新功能
```

> 先测试再开发至关重要——直接实现新功能可能在已破坏的基础上雪上加霜。

---

## 五、失败模式与对策总结

| 问题 | Initializer Agent | Coding Agent |
|------|-------------------|-------------|
| 过早宣布项目完成 | 建立功能清单 JSON | 每次启动先读清单，选一个功能 |
| 留下 Bug 或无文档进展 | 创建 git 仓库 + 进度文件 | 启动读进度 + git log，结束 commit + 更新进度 |
| 标记功能完成但没测试 | 建立功能清单 | 自我验证，测试通过才标记 passing |
| 花时间搞清如何运行应用 | 写 init.sh 脚本 | 启动时读 init.sh |

---

## 六、未来方向

- **多 Agent 架构**：测试 Agent、QA Agent、代码清理 Agent 可能比单一通用 Agent 更好
- **泛化到其他领域**：科学研究、金融建模等长时 Agent 任务

---

## 七、核心启示

1. **Compaction 不够，需要结构化的跨窗口记忆**——进度文件 + git 历史 + 功能清单
2. **每次只做一个功能**——增量进展是解决"一次性做太多"的关键
3. **用 JSON 而非 Markdown 存储结构化数据**——模型更不容易不当修改
4. **每次会话结束要留下"干净状态"**——可合并的代码、描述性 commit、进度更新
5. **先测试再开发**——启动时先验证应用正常，再实现新功能
6. **端到端测试必须像人类用户一样**——浏览器自动化 > 单元测试 > curl
7. **灵感来自人类工程师的日常实践**——好的 Agent 实践就是好的工程实践

---

## 参考资源

- [Quickstart 代码示例](https://github.com/anthropics/claude-quickstarts/tree/main/autonomous-coding)
- [Claude Agent SDK 文档](https://platform.claude.com/docs/en/agent-sdk/overview)
- [Claude 4 Prompting Guide - Multi-context Window Workflows](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices#multi-context-window-workflows)
