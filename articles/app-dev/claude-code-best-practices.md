# Claude Code 最佳实践：从环境配置到多会话并行

> 原文链接：[Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices)
>
> 日期：2025-10-01
>
> 作者：Anthropic
>
> 主题：claude-code, app-dev, context-engineering, workflow
>
> 类型：article

---

## 一、核心约束：上下文窗口

所有最佳实践的基础：**上下文窗口会快速填满，性能随之下降。**

上下文中包含整个对话、每个读取的文件、每个命令输出。一次调试或代码探索可能消耗数万 Token。

---

## 二、最高杠杆：让 Claude 自我验证

| 策略 | 反面 | 正面 |
|------|------|------|
| 提供验证标准 | "实现一个邮箱验证函数" | "写 validateEmail 函数。测试用例：user@example.com=true, invalid=false。实现后运行测试" |
| 视觉验证 UI | "让仪表盘更好看" | "[贴截图] 实现这个设计。截图结果对比原图，列出差异并修复" |
| 解决根因 | "构建失败了" | "构建失败报这个错：[贴错误]。修复并验证通过。解决根因，别压制错误" |

> 验证可以是测试套件、linter、Bash 命令。投资让你的验证无懈可击。

---

## 三、四阶段工作流

```
1. 探索（Plan Mode）→ 只读模式，读文件问问题
2. 计划 → 让 Claude 创建详细实现方案，Ctrl+G 可直接编辑计划
3. 实现（Normal Mode）→ 写代码、跑测试、修 Bug
4. 提交 → commit + 开 PR
```

何时跳过计划：能一句话描述 diff 的简单改动（修 typo、加日志、重命名变量）。
何时需要计划：不确定方法、修改多文件、不熟悉被修改的代码。

---

## 四、提示词技巧

| 策略 | 反面 | 正面 |
|------|------|------|
| 限定范围 | "给 foo.py 加测试" | "给 foo.py 写测试，覆盖用户未登录的边缘情况。避免 mock" |
| 指向来源 | "ExecutionFactory 的 API 为什么这么奇怪？" | "看 ExecutionFactory 的 git 历史，总结 API 如何演变" |
| 引用现有模式 | "加个日历组件" | "看首页现有组件的实现模式。HotDogWidget.php 是好例子。按模式实现日历组件" |
| 描述症状 | "修复登录 Bug" | "用户报告会话超时后登录失败。检查 src/auth/ 的 token 刷新。写失败测试复现后修复" |

提供富内容的方式：
- `@` 引用文件
- 直接贴图/截图
- 给 URL 提供文档引用
- `cat error.log | claude` 管道输入
- 让 Claude 自己拉取上下文

### 让 Claude 面试你

对于大型功能，先让 Claude 问你问题：

```
我想构建 [简要描述]。用 AskUserQuestion 工具详细面试我。
问技术实现、UI/UX、边缘情况、顾虑和权衡。
别问显而易见的问题，深入我可能没考虑到的难点。
持续面试直到全面覆盖，然后写完整规格到 SPEC.md。
```

面试完成后，**开新会话**执行——新会话有干净上下文，专注实现。

---

## 五、CLAUDE.md 编写指南

| 应该包含 | 不应该包含 |
|----------|------------|
| Claude 无法猜到的 Bash 命令 | 读代码就能看出的东西 |
| 与默认不同的代码风格规则 | Claude 已知的语言惯例 |
| 测试指令和首选测试工具 | 详细 API 文档（链接即可） |
| 仓库规范（分支命名、PR 约定） | 频繁变化的信息 |
| 项目特有的架构决策 | 冗长的教程和解释 |
| 开发环境的坑（必需环境变量） | 逐文件的代码库描述 |

关键原则：
- **保持精简**——每一行问"去掉它 Claude 会犯错吗？"不会就删
- 太长会让 Claude **忽略你的真正指令**
- 当规则被忽略 → 文件太长了；当 Claude 问已有答案的问题 → 措辞有歧义
- 用 "IMPORTANT" / "YOU MUST" 增加关键规则权重
- 提交到 git，团队共同维护，价值随时间复合

层级系统：
- `~/.claude/CLAUDE.md` → 全局所有项目
- `./CLAUDE.md` → 项目级（团队共享）
- `./CLAUDE.local.md` → 项目级个人（加入 .gitignore）
- 子目录 CLAUDE.md → Claude 按需拉取

支持 `@path/to/import` 语法导入其他文件。

---

## 六、环境扩展

| 扩展方式 | 用途 | 特点 |
|----------|------|------|
| **CLI 工具** | 与 GitHub/AWS/GCloud 交互 | 最高上下文效率 |
| **MCP Server** | 连接 Notion、Figma、数据库 | `claude mcp add` |
| **Hooks** | 每次编辑后自动跑 eslint 等 | 确定性，零例外 |
| **Skills** | 领域知识和可复用工作流 | `.claude/skills/`，按需加载 |
| **Subagents** | 独立上下文的专门助手 | `.claude/agents/`，安全审查等 |
| **Plugins** | 一键安装社区集成 | `/plugin` 浏览市场 |

---

## 七、上下文管理

| 操作 | 用途 |
|------|------|
| `/clear` | 不相关任务之间重置上下文 |
| `/compact [指令]` | 主动压缩，可指定保留重点 |
| `/rewind`（或 Esc+Esc） | 恢复到任意检查点（对话/代码/两者） |
| `/btw` | 侧边快速提问，不进入上下文历史 |
| 子 Agent | 研究探索在独立上下文完成 |
| `claude --continue` | 恢复最近会话 |
| `claude --resume` | 从历史会话中选择恢复 |

> 同一问题纠正两次还没对 → `/clear` + 更好的 prompt，几乎总是优于继续纠正。

---

## 八、自动化与规模化

### 非交互模式

```bash
claude -p "prompt"                              # 一次性查询
claude -p "prompt" --output-format json          # 结构化输出
claude -p "prompt" --output-format stream-json   # 流式输出
```

### 多会话并行

- **Writer/Reviewer 模式**：Session A 实现，Session B 审查
- **Test/Code 模式**：一个写测试，另一个写代码通过测试
- 新上下文审查避免对自己代码的偏见

### 大规模批处理

```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from React to Vue." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

先 2-3 个文件测试优化 prompt，再全量执行。

---

## 九、常见反模式

| 反模式 | 修复 |
|--------|------|
| 厨房水槽会话（混杂不相关任务） | `/clear` 隔离任务 |
| 反复纠正（同一问题 3+ 次） | `/clear` + 重写更好的初始 prompt |
| 过度膨胀的 CLAUDE.md | 无情裁剪，或转为 Hook |
| 信任而不验证 | 始终提供验证方式 |
| 无限探索（读数百文件） | 限定范围或用子 Agent |

---

## 十、核心启示

1. **上下文窗口是最重要的资源**——所有最佳实践都围绕管理这一有限资源
2. **自验证是最高杠杆操作**——给 Claude 测试、截图、预期输出来自我检查
3. **CLAUDE.md 像代码一样维护**——太长反而有害，定期裁剪和测试效果
4. **纠正两次就该重启**——干净上下文 + 更好的 prompt 胜过持续纠正
5. **子 Agent 是上下文管理利器**——研究探索放到独立上下文，主会话保持干净
6. **水平扩展**——多会话并行、非交互批处理、Writer/Reviewer 分离
7. **让 Claude 面试你**——大型功能先让 Claude 问问题，比你自己写规格更全面
8. **培养直觉**——知道何时详细何时开放，何时计划何时探索

---

## 参考资源

- [How Claude Code works](https://code.claude.com/en/how-claude-code-works)
- [Extend Claude Code](https://code.claude.com/en/features-overview)
- [Common workflows](https://code.claude.com/en/common-workflows)
- [CLAUDE.md 文档](https://code.claude.com/en/memory)
- [Context Window 详解](https://code.claude.com/en/context-window)
- [Permission Modes](https://code.claude.com/en/permission-modes)
- [Skills 文档](https://code.claude.com/en/skills)
- [Subagents 文档](https://code.claude.com/en/sub-agents)
- [Hooks 指南](https://code.claude.com/en/hooks-guide)
- [Sandboxing 文档](https://code.claude.com/en/sandboxing)
