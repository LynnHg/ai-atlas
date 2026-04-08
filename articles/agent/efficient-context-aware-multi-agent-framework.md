# 构建高效的上下文感知多 Agent 生产框架

> 原文链接：[Architecting efficient context-aware multi-agent framework for production](https://developers.googleblog.com/architecting-efficient-context-aware-multi-agent-framework-for-production/)
>
> 日期：2025-12-04
>
> 作者：Hangfei Lin（Google）
>
> 主题：agent, multi-agent, context-engineering
>
> 类型：article

## 上下文扩展的瓶颈

单纯依赖更大的上下文窗口无法解决生产问题，存在三大压力：

| 问题 | 表现 |
|------|------|
| **成本与延迟** | Token 成本和首 Token 延迟随上下文膨胀急剧上升 |
| **信号衰减** | "Lost in the middle"——无关信息淹没关键指令，模型决策质量下降 |
| **物理上限** | RAG 结果、中间产物、长对话最终溢出窗口 |

核心结论：增加 Token 只是拖延问题，必须改变上下文的**表示和管理方式**。

## 核心设计论点：上下文即编译视图

ADK 的核心理念：**上下文不是可变字符串缓冲区，而是从更丰富有状态系统编译出的视图**。

- **Session / Memory / Artifact** = 源（完整结构化状态）
- **Flow & Processors** = 编译管线（有序转换步骤）
- **Working Context** = 编译产物（送给 LLM 的单次调用视图）

三条设计原则：

1. **存储与呈现分离**：持久状态（Session）与按次调用视图（Working Context）独立演进
2. **显式变换**：通过命名、有序的 Processor 构建上下文，可观测、可测试
3. **默认最小范围**：每次模型调用只看到最小必要上下文，需要更多信息时通过工具主动获取

## 分层模型（Tiered Model）

| 层级 | 职责 | 生命周期 |
|------|------|----------|
| **Working Context** | 当前模型调用的即时 prompt | 临时，调用后丢弃 |
| **Session** | 交互的持久日志（Event 流） | 会话级持久化 |
| **Memory** | 跨会话长期知识（用户偏好、历史决策） | 长期持久化 |
| **Artifact** | 大型二进制/文本数据（文件、日志、图片） | 按名称和版本引用 |

### Working Context 是重新计算的视图

每次调用时，ADK 从底层状态重建 Working Context：先注入指令和身份，再拉取 Session 事件，可选附加 Memory 结果。这个视图是临时的、可配置的、模型无关的。

### Flow 与 Processor：上下文处理管线

ADK 中每个基于 LLM 的 Agent 都由一个 LLM Flow 支撑，维护有序的 Processor 列表：

```python
self.request_processors += [
    basic.request_processor,
    auth_preprocessor.request_processor,
    instructions.request_processor,
    identity.request_processor,
    contents.request_processor,
    context_cache_processor.request_processor,
    planning.request_processor,
    code_execution.request_processor,
    output_schema_processor.request_processor,
]
```

顺序很重要：每个 Processor 基于前一步的输出构建，提供了自然的插入点用于自定义过滤、压缩、缓存和多 Agent 路由。

### Session 与 Event：结构化历史

ADK 将每次交互捕获为强类型 Event 记录，而非原始 prompt 字符串，带来三个优势：

- **模型无关性**：可切换模型而无需重写历史
- **丰富操作**：压缩、时间旅行调试、记忆摄入可操作结构化事件流
- **可观测性**：精确检查状态转换和动作

`contents` Processor 负责将 Session 转换为 Working Context 中的历史部分，执行三步：**选择**（过滤无关事件）→ **转换**（展平为正确角色的 Content 对象）→ **注入**（写入 `llm_request.contents`）。

## 上下文压缩与缓存

### Context Compaction

达到可配置阈值后，ADK 触发异步流程：用 LLM 对滑动窗口内的旧事件做摘要，写回 Session 作为新的 "compaction" 事件，原始事件可降级或删除。

优势：Session 物理可管理 → `contents` Processor 自动工作于已压缩历史 → 压缩策略与 Agent 代码解耦。

### Context Filtering

确定性规则的预过滤插件，在到达模型前全局裁剪上下文。

### Context Caching（前缀缓存）

将上下文分为两个区域：

| 区域 | 内容 | 特点 |
|------|------|------|
| **稳定前缀** | 系统指令、Agent 身份、长期摘要 | 跨调用复用注意力计算 |
| **可变后缀** | 最新用户输入、新工具输出 | 每次变化 |

引入 `static instruction` 原语确保系统 prompt 不变，保持缓存前缀有效。这是全栈优化——从 prompt 结构设计到硬件张量计算复用。

## 相关性管理：人机协作

最优 Working Context 是人工领域知识与 Agent 动态决策的**协商结果**：

- **人类工程师**定义架构：数据存储位置、摘要方式、过滤规则
- **Agent** 提供智能：动态决定何时获取特定 Memory 或 Artifact

### Artifact：外化大型状态

避免"上下文倾倒"反模式——将 5MB CSV、大型 JSON 等直接塞入聊天历史。

ADK 采用**句柄模式**：大数据存于 Artifact Store，Agent 默认只看到轻量引用（名称和摘要）。需要时通过 `LoadArtifactsTool` 按需加载，任务完成后自动卸载。**5MB 噪音变成精确按需资源**。

### Memory：长期知识按需检索

`MemoryService` 两条设计原则：记忆必须可搜索（不固定嵌入）、检索应由 Agent 驱动。

两种召回模式：

| 模式 | 机制 |
|------|------|
| **反应式召回** | Agent 识别知识缺口，主动调用 `load_memory_tool` 搜索 |
| **主动式召回** | 预处理器基于最新用户输入做相似性搜索，调用前注入 |

## 多 Agent 上下文管理

### 两种交互模式

| 模式 | 特点 |
|------|------|
| **Agent as Tool** | 子 Agent 作为函数调用，只看到特定指令和必要 Artifact，无历史 |
| **Agent Transfer** | 完全交接控制权，子 Agent 继承 Session 视图，可驱动工作流 |

### Scoped Handoff（作用域交接）

通过 `include_contents` 控制上下文传递量：

- **full 模式**：传递调用者完整 Working Context（子 Agent 需要全部历史时）
- **none 模式**：子 Agent 不看历史，只接收新构建的 prompt（专用 Agent 最小上下文）

交接规则复用同一个 Flow 管线，无需额外多 Agent 机制层。

### 对话翻译

模型只理解 `system`/`user`/`assistant` 角色，不原生理解多 Agent。交接时 ADK 执行主动翻译：

- **叙事重塑（Narrative Casting）**：前任 Agent 的 "Assistant" 消息重写为叙事上下文（如 `[For context]: Agent B said...`）
- **动作归属（Action Attribution）**：其他 Agent 的工具调用被标记或摘要，防止新 Agent 混淆

## 核心启示

1. **上下文是架构问题**，不是 prompt 拼接——需要像系统工程一样设计存储、编译、缓存策略
2. **"编译器"心智模型**非常强大：源码（Session/Memory/Artifact）→ 编译管线（Processors）→ 目标代码（Working Context）
3. **默认最小上下文 + 按需获取**是生产级 Agent 的正确策略，避免"context dumping"反模式
4. **多 Agent 场景必须显式管理上下文边界**，否则 token 爆炸 + 角色混淆
5. **前缀缓存友好设计**是全栈优化的体现——从 prompt 结构到硬件计算复用

## 参考资源

- [Google Agent Development Kit (ADK)](https://github.com/google/adk-python)
