# 扩展 Managed Agents：将大脑与双手解耦

> 原文链接：[Scaling Managed Agents: Decoupling the brain from the hands](https://www.anthropic.com/engineering/managed-agents)
>
> 日期：2026-04-13
>
> 作者：Lance Martin, Gabe Cemaj, Michael Cohen（Anthropic）
>
> 主题：agent, managed-agents, architecture, context-engineering
>
> 类型：article

## 核心观点

Harness 编码的假设会随着模型能力提升而过时。Managed Agents 通过稳定的接口抽象，让底层实现可以自由演进。

**关键案例**：Claude Sonnet 4.5 会在接近上下文限制时出现"上下文焦虑"，提前结束任务。但 Claude Opus 4.5 已不存在这个问题，之前添加的上下文重置机制变成了死代码。

## 设计理念：借鉴操作系统

操作系统将硬件虚拟化为足够通用的抽象（进程、文件），使其能支持尚未被创造的程序。`read()` 命令不关心底层是 1970 年代的磁盘还是现代 SSD。

Managed Agents 遵循同样的模式，将 Agent 组件虚拟化：

| 组件 | 职责 | 特点 |
|------|------|------|
| **Session** | 记录所有事件的只追加日志 | 持久化存储在外部 |
| **Harness** | 调用 Claude 并路由工具调用 | 无状态，可随时重启 |
| **Sandbox** | 代码执行和文件编辑环境 | 独立隔离，可随时销毁重建 |

## 从"宠物"到"牲口"

### 旧架构的问题（耦合式设计）

最初将所有组件放在单个容器中：

- **调试困难**：只能通过 WebSocket 事件流观察，harness bug、网络丢包、容器离线表现一致
- **状态丢失**：容器失败 = 会话丢失（典型的"宠物"问题）
- **VPC 接入难**：假设所有资源都在同一环境，客户接入私有云很困难

### 新架构的解决方案

**Harness 离开容器**：

- Sandbox 调用方式：`execute(name, input) → string`
- 容器死亡 → harness 捕获工具调用错误 → 返回给 Claude → Claude 决定是否重试
- 新容器初始化：`provision({resources})`

**Harness 故障恢复**：

- Session 日志在 harness 外部，crash 不丢失状态
- 恢复流程：`wake(sessionId)` → `getSession(id)` → 从最后事件恢复
- 运行时：`emitEvent(id, event)` 持久记录事件

## 安全边界设计

**核心问题**：耦合设计中，Claude 生成的不可信代码与凭据在同一容器。Prompt injection 只需说服 Claude 读取环境变量即可获取 token，然后生成不受限的新会话。

**解决方案**：让 token 永远无法从沙箱内访问。

| 资源类型 | 安全方案 |
|----------|----------|
| **Git 仓库** | 初始化时用 token 克隆，配置到本地 remote，沙箱内 push/pull 无需接触 token |
| **MCP 工具** | OAuth token 存储在安全 vault，通过专用代理访问，harness 永远不知道凭据 |

## Session ≠ Claude 的上下文窗口

长时任务超出上下文窗口时，传统方案（压缩、裁剪、记忆文件）都涉及**不可逆的决策**——很难预判未来哪些 token 有用。

**Managed Agents 方案**：

```
┌─────────────────────────────────────────────────┐
│                 Session Log                      │
│  (持久化存储，保留完整历史)                         │
└──────────────────────┬──────────────────────────┘
                       │ getEvents()
                       ▼
┌─────────────────────────────────────────────────┐
│                   Harness                        │
│  (context 变换：裁剪、压缩、缓存优化)              │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│           Claude 上下文窗口                       │
│  (有限容量，经过优化的 context)                    │
└─────────────────────────────────────────────────┘
```

- Session 日志作为**独立于上下文窗口的持久化对象**
- `getEvents()` 接口允许灵活访问：位置切片、回溯、重读
- Harness 负责在传入 Claude 前进行**任意变换**（上下文工程、缓存优化等）

关注点分离：可恢复的上下文存储（Session）与任意上下文管理（Harness）分开，因为无法预测未来模型需要什么样的上下文工程。

## 多大脑 × 多双手

### 多大脑（Many Brains）

将 harness 从容器中移出后的性能提升：

- **p50 TTFT 下降约 60%**
- **p95 TTFT 下降超过 90%**

原因：不需要沙箱的会话无需等待容器初始化，推理可立即开始。扩展多大脑只需启动多个无状态 harness，按需连接手。

### 多双手（Many Hands）

- 每个手都是工具接口：`execute(name, input) → string`
- Harness 不关心沙箱是容器、手机还是 Pokémon 模拟器
- 手与大脑解耦后，大脑之间可以**传递手**

早期模型无法处理多执行环境的认知任务，单容器是合理选择。随着智能提升，单容器反而成为限制——容器失败会丢失所有手的状态。

## 设计哲学总结

Managed Agents 是**"元编排器"（Meta-harness）**：

| 层次 | 态度 |
|------|------|
| **接口形状** | 有明确立场（Session、Sandbox、多大脑多双手） |
| **具体实现** | 不做假设（任何 harness 都可插入） |
| **时间维度** | 为"尚未被想到的程序"设计 |

接口设计的核心假设：

- Claude 需要操作状态（Session）
- Claude 需要执行计算（Sandbox）
- Claude 需要扩展到多大脑和多双手
- 这些能力需要在长时间范围内可靠、安全地运行

不做假设的部分：Claude 需要多少大脑和手，以及它们在哪里。

## 核心启示

1. **Harness 假设会过时**：模型能力提升后，之前的补丁可能变成负担，需要持续审视
2. **解耦是可扩展性的关键**：将有状态组件（Session）与无状态组件（Harness）分离
3. **安全需要结构性保障**：凭据永远不应进入代码执行环境，即使模型"变聪明"也无法获取
4. **Context 管理的两层分离**：持久存储（Session）与动态管理（Harness）分开，保留恢复能力
5. **接口设计优于实现设计**：为未来的模型和 harness 留出演进空间

## 相关资源

- [构建高效 AI Agent](https://www.anthropic.com/engineering/building-effective-agents) — Anthropic Agent 设计基础
- [长时运行 Agent 的有效编排](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Harness 设计详解
- [高效的 AI Agent 上下文工程](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — 上下文管理技术
- [Managed Agents 文档](https://platform.claude.com/docs/en/managed-agents/overview) — 官方使用文档
- [Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html) — 关于假设过时的经典文章
