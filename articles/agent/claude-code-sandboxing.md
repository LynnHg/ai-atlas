# Claude Code 沙箱安全：让 Agent 更安全也更自主

> 原文链接：[Beyond permission prompts: making Claude Code more secure and autonomous](https://www.anthropic.com/engineering/claude-code-sandboxing)
>
> 日期：2025-10-20
>
> 作者：David Dworken, Oliver Weller-Davies
>
> 主题：agent, security, sandboxing, claude-code
>
> 类型：article

---

## 一、核心问题：自主性 vs 安全性的矛盾

Claude Code 需要读写文件、运行命令、调试代码，但这种高权限带来风险：

| 问题 | 说明 |
|------|------|
| **权限疲劳** | 默认每个操作都要点"允许"，频繁审批导致用户不再仔细看内容 |
| **提示注入风险** | 恶意内容可能诱导 Claude 修改系统文件或泄露敏感信息 |

> 沙箱启用后，权限提示减少了 **84%**。

核心思路：**不再逐个审批，而是预定义安全边界**——在边界内自由工作，越界才需确认。

---

## 二、沙箱的核心设计：两道隔离边界

沙箱不是单一技术，而是**文件系统隔离 + 网络隔离**的组合：

| 隔离维度 | 作用 | 防御场景 |
|----------|------|----------|
| **文件系统隔离** | Claude 只能访问/修改指定目录 | 防止修改 SSH 密钥、系统文件等 |
| **网络隔离** | Claude 只能连接已批准的服务器 | 防止数据泄露、下载恶意软件 |

> 关键认知：**两者缺一不可**。没有网络隔离，被攻破的 Agent 可以把 SSH 密钥发送出去；没有文件系统隔离，Agent 可以轻松逃逸沙箱获取网络权限。

---

## 三、技术实现

### 3.1 沙箱化 Bash 工具

基于操作系统级原语构建：
- **Linux**：使用 [bubblewrap](https://github.com/containers/bubblewrap)
- **macOS**：使用 seatbelt
- 覆盖 Claude Code 的直接交互 + **所有被派生的脚本、程序和子进程**

文件系统隔离：
- 允许读写当前工作目录
- 阻止修改工作目录外的任何文件

网络隔离：
- 所有网络访问必须通过 Unix Domain Socket 连接到沙箱外的**代理服务器**
- 代理服务器强制执行域名限制
- 新域名请求时通知用户确认
- 支持自定义代理规则，对出站流量执行任意过滤策略

两个维度都可配置：可自定义允许/禁止的文件路径和网络域名。

### 3.2 Claude Code on the Web

在云端隔离沙箱中运行 Claude Code：
- 每个会话一个独立沙箱
- **敏感凭证（Git 凭证、签名密钥）从不进入沙箱**

Git 安全代理工作流程：

```
Claude Code（沙箱内）
  → 使用受限范围凭证发起 Git 请求
    → 代理服务器验证凭证和操作内容
      → 检查是否只推送到配置的分支
        → 附加真实认证 Token
          → 转发到 GitHub
```

---

## 四、沙箱的权限模型

```
沙箱内操作 → 自动允许（无需确认）
   ↓ 如果尝试越界
尝试访问沙箱外资源 → 立即通知用户 → 用户选择允许或拒绝
```

即使提示注入成功，攻击也被**完全隔离**在沙箱内，无法影响用户整体安全。

---

## 五、核心启示

1. **沙箱 = 预定义安全边界**——在边界内自由工作，越界才需确认，比逐个审批更安全也更高效
2. **文件系统隔离 + 网络隔离缺一不可**——单一维度的隔离都有致命逃逸路径
3. **基于 OS 级原语构建**——不是应用层模拟，而是用 bubblewrap/seatbelt 等内核级能力
4. **凭证与执行环境分离**——敏感凭证永远不进入 Agent 能触及的沙箱
5. **代理服务器模式处理网络访问**——所有出站流量经过验证和过滤
6. **权限疲劳是真实的安全风险**——频繁审批反而降低安全性，沙箱减少 84% 提示
7. **已开源**——其他团队可以复用这套沙箱技术来增强自己 Agent 的安全

---

## 参考资源

- [Claude Code 沙箱文档](https://docs.claude.com/en/docs/claude-code/sandboxing)
- [沙箱运行时开源仓库](https://github.com/anthropic-experimental/sandbox-runtime)
- [bubblewrap（Linux 沙箱）](https://github.com/containers/bubblewrap)
- [Claude Code on the Web 文档](https://docs.claude.com/en/docs/claude-code/claude-code-on-the-web)
- [Claude Code on the Web 发布博客](https://www.anthropic.com/news/claude-code-on-the-web)
