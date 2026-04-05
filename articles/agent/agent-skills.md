# Agent Skills：用文件和文件夹装备 Agent

> 原文链接：[Equipping agents for the real world with Agent Skills](https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills)
>
> 日期：2025-10-16
>
> 作者：Barry Zhang, Keith Lazuka, Mahesh Murag
>
> 主题：agent, skills, progressive-disclosure, context-engineering
>
> 类型：article

---

## 一、什么是 Agent Skills

Agent Skills = 一个包含 `SKILL.md` 的**目录**，里面组织了指令、脚本和资源，Agent 可以动态发现和加载。

核心类比：

> 为 Agent 构建 Skill 就像为新员工准备入职指南——不是为每个场景定制独立的 Agent，而是用**可组合的知识包**将通用 Agent 特化。

Skills 已发布为[开放标准](https://agentskills.io/)，支持跨平台可移植（2025-12-18 更新）。

---

## 二、Skill 的结构解剖

### 2.1 最小结构

```
skill-name/
├── SKILL.md          # 必需：主指令文件
├── reference.md      # 可选：详细参考资料
├── forms.md          # 可选：特定场景指令
└── scripts/          # 可选：可执行脚本
    └── extract.py
```

### 2.2 SKILL.md 的构成

必须以 YAML frontmatter 开头，包含两个必需字段：

| 字段 | 作用 |
|------|------|
| `name` | 唯一标识符，Agent 启动时预加载 |
| `description` | Agent 判断何时触发该 Skill 的依据 |

正文包含具体的指令和引用的附加文件。

---

## 三、核心设计原则：渐进式披露（Progressive Disclosure）

这是 Skills 设计的灵魂——像一本好手册：先看目录，再看章节，最后查附录。

| 层级 | 内容 | 加载时机 |
|------|------|----------|
| **第 1 层** | `name` + `description`（元数据） | Agent 启动时全部预加载到系统提示词 |
| **第 2 层** | `SKILL.md` 完整正文 | Agent 判断 Skill 与当前任务相关时读取 |
| **第 3 层+** | 引用的附加文件（reference.md、forms.md 等） | Agent 在执行过程中按需读取 |

关键优势：
- Agent **不需要**一次性将整个 Skill 读入上下文
- Skill 中可以捆绑的上下文量**实际上不受限制**
- 保持核心 SKILL.md 精简，Agent 只在需要时深入

---

## 四、Skill 与上下文窗口的交互流程

```
1. 启动
   → 上下文中有：系统提示词 + 所有 Skill 的 name/description + 用户消息

2. Agent 判断需要 PDF Skill
   → 调用 Bash 读取 pdf/SKILL.md

3. Agent 发现需要填表功能
   → 按需读取 forms.md

4. Agent 拥有足够上下文
   → 执行用户任务
```

---

## 五、Skill 与代码执行

LLM 擅长很多事，但某些操作**用代码更高效**（如排序列表、数据提取等确定性操作）。

Skill 可以包含预写的脚本：
- Agent 在需要时**执行脚本**，而非将脚本或数据加载到上下文
- 代码执行是**确定性**的，结果一致且可重复
- 比让 LLM 生成代码更可靠、更省 Token、更省时间

> 需要在 Skill 中明确标注 Agent 应该**运行**脚本还是**阅读**脚本作为参考。

---

## 六、开发和评估 Skill 的最佳实践

| 原则 | 具体做法 |
|------|----------|
| **从评估开始** | 先跑代表性任务，观察 Agent 在哪里挣扎，再针对性构建 Skill |
| **为规模而结构化** | SKILL.md 过长时拆分为独立文件；互斥或低频内容分离以减少 Token |
| **站在 Claude 的角度思考** | 监控 Agent 如何使用 Skill，关注意外轨迹或过度依赖；特别注意 name 和 description |
| **与 Claude 迭代** | 让 Claude 捕获成功方法和常见错误写入 Skill；出错时让 Claude 自我反思 |

> 关键洞察：**让 Claude 发现它实际需要什么上下文**，而非提前猜测。

---

## 七、安全考量

Skill 提供了指令和代码两种能力，恶意 Skill 可能：
- 在运行环境中引入漏洞
- 指导 Agent 窃取数据或执行意外操作

建议：
- 只从**可信来源**安装 Skill
- 不可信来源的 Skill 需要**完整审计**：代码依赖、捆绑资源、外部网络连接指令

---

## 八、未来方向

- 支持 Skill 的完整生命周期：创建、编辑、发现、分享、使用
- 与 MCP 互补：Skill 教 Agent 复杂工作流，MCP 提供外部工具接入
- 终极目标：**Agent 自己创建、编辑和评估 Skill**，将行为模式固化为可复用能力

---

## 九、核心启示

1. **Skill = 新员工入职指南**——用文件和文件夹为 Agent 注入领域专业能力
2. **渐进式披露是核心设计原则**——元数据 → 主文件 → 附加文件，逐层加载
3. **保持 SKILL.md 精简**——详细内容放附加文件，Agent 按需读取
4. **代码脚本比 LLM 生成更可靠**——确定性操作用预写脚本
5. **从评估开始，与 Claude 迭代**——观察失败模式再构建，而非提前猜测需求
6. **Skill 是可组合的**——不需要为每个场景造独立 Agent，组合 Skill 即可特化
7. **Agent 未来能自己积累 Skill**——行为模式固化为可复用能力，能力随时间增长

---

## 参考资源

- [Agent Skills 开放标准](https://agentskills.io/)
- [Agent Skills 文档](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [Agent Skills Cookbook](https://github.com/anthropics/claude-cookbooks/tree/main/skills)
- [官方 Skills 仓库](https://github.com/anthropics/skills)
- [PDF Skill 示例](https://github.com/anthropics/skills/tree/main/document-skills/pdf)
- [MCP 协议官网](https://modelcontextprotocol.io/)
