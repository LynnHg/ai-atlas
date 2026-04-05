# 代码执行 + MCP：构建更高效的 Agent

> 原文链接：[Code execution with MCP: Building more efficient agents](https://www.anthropic.com/engineering/code-execution-with-mcp)
>
> 日期：2025-11-04
>
> 作者：Adam Jones, Conor Kelly
>
> 主题：mcp, code-execution, agent, context-engineering, tool-use
>
> 类型：article

---

## 一、传统 MCP 工具调用的两大问题

### 1.1 工具定义占满上下文

大多数 MCP 客户端在启动时将所有工具定义加载到上下文。当连接数千个工具时，模型在读到用户请求前就已经处理了**数十万 Token** 的工具描述。

### 1.2 中间结果反复穿过上下文

传统方式中，每次工具调用的结果都必须流经模型上下文。

案例——将 Google Drive 会议纪要附加到 Salesforce 线索：
- 第一次：`gdrive.getDocument` 返回完整文稿 → 进入上下文
- 第二次：`salesforce.updateRecord` 需要模型把整份文稿重新写入调用参数 → 文稿又穿过上下文一次

> 一场 2 小时的会议纪要，可能额外消耗 **50,000+ Token**，大文档甚至会超出上下文窗口限制。模型在复制大量数据时也更容易出错。

---

## 二、核心方案：将 MCP 工具暴露为代码 API

### 2.1 核心思路

不再让模型逐个"调用"工具，而是将 MCP Server 的工具生成为**代码文件树**，让 Agent 通过写代码来交互：

```
servers/
├── google-drive/
│   ├── getDocument.ts
│   └── index.ts
├── salesforce/
│   ├── updateRecord.ts
│   └── index.ts
└── ...
```

每个工具文件包含类型定义和调用函数：

```typescript
// ./servers/google-drive/getDocument.ts
interface GetDocumentInput {
  documentId: string;
}

interface GetDocumentResponse {
  content: string;
}

export async function getDocument(input: GetDocumentInput): Promise<GetDocumentResponse> {
  return callMCPTool<GetDocumentResponse>('google_drive__get_document', input);
}
```

Agent 通过浏览文件系统发现工具，读取需要的工具文件了解接口，然后写代码调用。

### 2.2 效果对比

同样是"读文稿写入 Salesforce"任务：

**代码执行方式**：

```typescript
const transcript = (await gdrive.getDocument({ documentId: 'abc123' })).content;
await salesforce.updateRecord({
  objectType: 'SalesMeeting',
  recordId: '00Q5f000001abcXYZ',
  data: { Notes: transcript }
});
```

数据在代码执行环境中直接传递，不经过模型上下文。

| 对比项 | 传统方式 | 代码执行方式 |
|--------|----------|-------------|
| Token 消耗 | 150,000 | 2,000 |
| 节省比例 | - | **98.7%** |
| 数据流经模型 | 每次工具调用都穿过 | 仅最终结果 |

Cloudflare 发布了类似发现，称之为 "Code Mode"。

---

## 三、五大核心优势

### 3.1 渐进式发现（Progressive Disclosure）

- 模型通过浏览文件系统**按需加载**工具定义，而非一次性全部加载
- 可提供 `search_tools` 工具，支持按名称/描述搜索
- 支持 detail level 参数控制返回详情等级（仅名称 / 名称+描述 / 完整 Schema）

### 3.2 上下文高效的数据处理

在代码环境中过滤和转换数据，而非让模型处理全量数据：

```typescript
const allRows = await gdrive.getSheet({ sheetId: 'abc123' });
const pendingOrders = allRows.filter(row => row["Status"] === 'pending');
console.log(`Found ${pendingOrders.length} pending orders`);
console.log(pendingOrders.slice(0, 5)); // 只返回前 5 行
```

| 场景 | 传统方式 | 代码执行方式 |
|------|----------|-------------|
| 10,000 行表格 | 全部进入上下文 | 代码过滤后只返回 5 行 |
| 多源数据聚合 | 每个结果都穿过上下文 | 代码中 join，只返回摘要 |

### 3.3 更强大的控制流

循环、条件、错误处理用代码实现，不再逐次工具调用：

```typescript
let found = false;
while (!found) {
  const messages = await slack.getChannelHistory({ channel: 'C123456' });
  found = messages.some(m => m.text.includes('deployment complete'));
  if (!found) await new Promise(r => setTimeout(r, 5000));
}
console.log('Deployment notification received');
```

比交替执行"MCP 工具调用 → 睡眠 → 再调用"高效得多，还能减少 TTFT（首 Token 延迟）。

### 3.4 隐私保护

中间结果**默认留在代码执行环境**，不进入模型上下文。更敏感的场景可以在 MCP 客户端层自动 Token 化 PII：

```typescript
// 模型看到的（已脱敏）：
[
  { salesforceId: '00Q...', email: '[EMAIL_1]', phone: '[PHONE_1]', name: '[NAME_1]' },
  { salesforceId: '00Q...', email: '[EMAIL_2]', phone: '[PHONE_2]', name: '[NAME_2]' }
]
```

- 真实数据在 MCP 工具间直接传递，不穿过模型
- 工具调用时自动还原真实值
- 可定义确定性安全规则控制数据流向

### 3.5 状态持久化与技能复用

Agent 可以将中间结果写入文件，跨执行恢复状态。更进一步，Agent 可以将有效代码保存为**可复用的技能（Skills）**：

```typescript
// ./skills/save-sheet-as-csv.ts
export async function saveSheetAsCsv(sheetId: string) {
  const data = await gdrive.getSheet({ sheetId });
  const csv = data.map(row => row.join(',')).join('\n');
  await fs.writeFile(`./workspace/sheet-${sheetId}.csv`, csv);
  return `./workspace/sheet-${sheetId}.csv`;
}
```

配合 SKILL.md 文件，形成结构化的技能库，Agent 能力随时间不断增长。

---

## 四、权衡与注意事项

代码执行不是免费的午餐：

| 收益 | 成本 |
|------|------|
| Token 成本降低 98.7% | 需要安全的沙箱执行环境 |
| 延迟改善 | 需要资源限制和监控 |
| 工具组合能力增强 | 增加运维复杂度 |
| 隐私保护 | 安全审计需求 |

> 需要在效率提升与实现复杂度之间做权衡。

---

## 五、核心启示

1. **MCP 工具作为代码 API > 直接工具调用**——Token 节省可达 98.7%
2. **让数据在代码环境中流转，而非穿过模型上下文**——这是效率提升的根本来源
3. **渐进式发现是管理大量工具的关键模式**——按需加载而非全量预加载
4. **代码执行天然支持隐私保护**——中间数据不进入模型，PII 可自动脱敏
5. **Agent 可以积累技能**——保存有效代码为可复用函数，能力随时间增长
6. **传统软件工程的解法适用于 Agent**——上下文管理、工具组合、状态持久化都有成熟方案

---

## 参考资源

- [MCP 协议官网](https://modelcontextprotocol.io/)
- [MCP Server 仓库](https://github.com/modelcontextprotocol/servers)
- [MCP SDK 文档](https://modelcontextprotocol.io/docs/sdk)
- [Cloudflare Code Mode](https://blog.cloudflare.com/code-mode/)
- [Claude Code Sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing)
- [Agent Skills 文档](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [MCP 社区](https://modelcontextprotocol.io/community/communication)
