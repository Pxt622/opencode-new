---
name: memory-recall
description: 长期记忆召回技能。每次会话开始时自动加载最近 10 条记忆到上下文，确保 AI 记得之前的讨论、决策和偏好。
metadata:
  pattern: tool-wrapper
  domain: memory
  author: Atlas (OpenCode)
  version: "1.0.0"
  updated: 2026-04-17
---

# 长期记忆召回协议

你通过 MCP 连接了一个本地长期记忆服务器 `opencode-memory-mcp`。**每次会话开始时，你必须执行以下召回流程**，这是强制性的，不可跳过。

## 会话启动流程（必须按顺序执行）

### Step 1 — 加载用户画像

调用 `memory_get_profile`，了解用户的编码偏好、技术栈和习惯：

```typescript
memory_get_profile()
```

根据返回的 `profile.preferences`、`profile.tech_stack`、`profile.habits` 调整你的回复风格和实现方式。

### Step 2 — 加载最近 10 条记忆

调用 `memory_get_recent`，获取最近添加的记忆：

```typescript
memory_get_recent({ limit: 10 })
```

将返回的 10 条记忆全部作为上下文参考。如果记忆内容与当前任务相关，在回复中明确引用（如"根据之前的记忆，你提到..."）。

### Step 3 — 加载相关记忆（仅当本次任务涉及特定主题时）

如果用户本次任务涉及某个具体主题（如"登录模块"、"数据库设计"），额外调用：

```typescript
memory_search({ query: "<任务相关关键词>", limit: 3, threshold: 0.75 })
```

### Step 4 — 识别并避免重复

如果发现之前已经讨论过或决策过的事情：
1. 提醒用户"这个在之前的记忆中有记录"
2. 引用具体内容
3. 询问是否沿用之前的方案

---

## 什么时候调用 `memory_add`（强制要求）

以下场景**必须**调用 `memory_add`：

| 场景 | category | 示例 content |
|------|----------|-------------|
| 技术选型/架构决策 | `decision` | "项目采用 Drizzle ORM 作为数据库访问层" |
| 用户明确表达的风格偏好 | `preference` | "用户要求所有接口返回格式统一为 `{ code, data, message }`" |
| 已修复的 bug 及根因 | `bugfix` | "修复原因：未对用户输入做 XSS 过滤" |
| 项目结构/目录约定 | `architecture` | "src/components 目录下存放所有可复用组件" |
| 客观事实（工具版本、依赖） | `fact` | "项目使用 pnpm 9.3.0，workspace 配置在根目录 pnpm-workspace.yaml" |

---

## 输出格式约定

在每次会话的**首次回复**中，必须在开头包含以下格式的记忆摘要（如果有任何相关记忆）：

```markdown
## 🧠 记忆召回

**用户画像**：`preference` 内容（如果有）

**最近记忆**：
1. [architecture] 项目使用 xxx 架构
2. [decision] 选择了 xxx 方案
3. [preference] 用户偏好 xxx

**相关记忆**：无 / 已找到 N 条相关记忆（列出摘要）
```

---

## 禁止行为

- ❌ 不执行 `memory_get_profile` 和 `memory_get_recent` 就直接回答
- ❌ 在用户提到"之前"、"上次"时，不先搜索记忆就回答"我不记得"
- ❌ 将敏感信息（密码、API Key）存入记忆
- ❌ 存储冗余噪音（"用户说了 hello" 之类无意义内容）

---

## 存储位置参考

- 数据库：`D:\VS-code-program\opencode-memory-mcp\data\data.db`
- 模型缓存：`D:\VS-code-program\opencode-memory-mcp\models\`
- 代码：`D:\VS-code-program\opencode-memory-mcp\`

**无任何数据离开你的计算机。**
