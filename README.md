# OpenCode Config & Skills

> My complete OpenCode configuration — all skills, MCP servers, and settings.
> Clone once, deploy anywhere. All paths are relative, no secrets included.

---

## 📦 What's Included

### 1. `opencode-memory-mcp/`

**本地长期记忆 MCP 服务器**，为 AI coding agent 提供跨会话记忆能力。

- **存储**：SQLite 本地数据库，完全离线
- **向量搜索**：本地嵌入模型 `all-MiniLM-L6-v2`（首次运行自动下载 ~23MB）
- **5 个工具**：`memory_add`、`memory_search`、`memory_get_recent`、`memory_delete`、`memory_get_profile`
- **去重**：相似度 >0.92 视为重复，自动跳过存储
- **复刻路径**：`D:\VS-code-program\opencode-memory-mcp\`（代码 + 数据 + 模型全在项目内）

### 2. `.opencode/`

OpenCode 的全部配置和技能：

| 文件/目录 | 说明 |
|-----------|------|
| `opencode.json` | 主配置（providers、MCP servers、plugins） |
| `oh-my-openagent.json` | oh-my-openagent 插件配置 |
| `auth.json` | ⚠️ **不上传**（含 API Key）— 复刻时需自行配置 |
| `command/` | OpenCode 命令定义（opsx-apply、opsx-propose 等） |
| `skills/` | 所有 skills（~50+ 个子目录，含参考资料、脚本、数据） |

### 3. `AGENTS.md`

项目根目录的开发协议文件，定义通用开发规范。

---

## 🚀 快速复刻

### 前置条件

- **Node.js** 18+
- **npm** 10+
- **Python** 3.10+（部分 skill 需要）
- **Git**

### Step 1 — 克隆仓库

```bash
git clone https://github.com/<your-username>/opencode-config.git
cd opencode-config
```

### Step 2 — 安装 MCP 服务器依赖

```bash
cd opencode-memory-mcp
npm install
npm run build
cd ..
```

### Step 3 — 配置 OpenCode

1. 将 `.opencode` 目录复制到你的用户目录：

```powershell
# Windows PowerShell
Copy-Item -Path ".opencode" -Destination "$HOME/.opencode" -Recurse -Force

# Linux/macOS
cp -r .opencode ~/.opencode/
```

2. **编辑 `~/.opencode/auth.json`**，填入你的 API Key：

```json
{
  "providers": {
    "bailian": {
      "apiKey": "YOUR_BAILIAN_API_KEY"
    }
  }
}
```

3. **编辑 `~/.opencode/opencode.json`**：
   - 检查 `mcpServers.memory.args` 中的路径是否需要更新（指向你的克隆路径）
   - 确认 `plugin` 配置是否需要调整

### Step 4 — 启动 OpenCode

```bash
opencode
```

---

## 🧠 MCP 服务器使用方式

### 调用方式

```typescript
// 添加记忆（自动去重）
memory_add({ content: "项目使用 Drizzle ORM", category: "architecture" })

// 语义搜索
memory_search({ query: "数据库方案", limit: 5, threshold: 0.75 })

// 获取最近 10 条
memory_get_recent({ limit: 10 })

// 删除记忆
memory_delete({ memory_id: "mem_xxx" })

// 获取用户画像
memory_get_profile()
```

### Skill 自动召回

`skills/memory-recall/SKILL.md` 定义了每次会话开始时**自动加载最近 10 条记忆**的协议。AI 会自动执行：

```
memory_get_profile() → memory_get_recent(10) → memory_search(相关主题)
```

---

## 📁 仓库结构

```
opencode-config/
├── README.md
├── .gitignore
├── AGENTS.md
│
├── opencode-memory-mcp/              # MCP 服务器（TypeScript）
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── index.ts                  # 入口
│   │   ├── server.ts                 # MCP 服务器 + 5 个工具
│   │   ├── db.ts                     # SQLite + brute-force 向量搜索
│   │   ├── embeddings.ts             # 本地 all-MiniLM-L6-v2 嵌入
│   │   └── types.ts                  # 类型定义
│   ├── data/                         # SQLite 数据库（首次运行自动创建）
│   └── models/                       # 嵌入模型缓存（首次运行自动下载）
│
└── .opencode/                       # OpenCode 主配置
    ├── opencode.json                 # 主配置
    ├── oh-my-openagent.json
    ├── auth.json                     # ⚠️ 不上传，需自行配置
    ├── package.json
    ├── command/                      # 命令定义
    │   ├── opsx-apply.md
    │   ├── opsx-archive.md
    │   ├── opsx-explore.md
    │   └── opsx-propose.md
    └── skills/                       # 所有 skills (~50+)
        ├── memory-recall/            # ⭐ 长期记忆召回
        ├── harmony-arkts/            # 鸿蒙 ArkTS 开发
        ├── mcp-builder/              # MCP 服务器构建
        ├── design-system/
        ├── frontend-design/
        ├── doc-coauthoring/
        ├── docx/
        ├── pdf/
        ├── pptx/
        ├── slides/
        ├── xlsx/
        ├── ui-styling/
        ├── web-access/
        └── ...（更多 skills）
```

---

## 🔧 常见问题

### Q: 首次运行报 `better-sqlite3` 编译错误？

```bash
# Windows 需要 C++ 构建工具
npm install --build-from-source better-sqlite3

# 或者使用 electron-rebuild
npm install -g @electron/rebuild
npx electron-rebuild -f -w better-sqlite3
```

### Q: `memory_search` 第一次调用很慢？

正常现象。首次调用时 `@xenova/transformers` 会从 HuggingFace 下载模型（约 23MB），之后完全离线。

### Q: 如何迁移已有记忆？

```powershell
# 复制整个 data 目录
Copy-Item -Path "D:\VS-code-program\opencode-memory-mcp\data" -Destination "NEW_PATH\opencode-memory-mcp\data" -Recurse

# 复制模型缓存（加速首次运行）
Copy-Item -Path "D:\VS-code-program\opencode-memory-mcp\models" -Destination "NEW_PATH\opencode-memory-mcp\models" -Recurse
```

### Q: `auth.json` 包含哪些字段需要填写？

参考 `.opencode/opencode.json` 中的 `provider` 配置，填入对应 API Key：
- `bailian` → 阿里云百炼 API Key
- `deepseek` → DeepSeek API Key
- `ollama` → 本地 Ollama（无需 Key）

---

## 📝 自定义修改

### 添加新的 Skill

在 `~/.opencode/skills/` 下创建新目录，包含 `SKILL.md` 即可被 OpenCode 自动加载。

### 修改 MCP 服务器配置

编辑 `~/.opencode/opencode.json` 中的 `mcpServers` 节点：

```json
"mcpServers": {
  "memory": {
    "command": "node",
    "args": ["D:\\你的路径\\opencode-memory-mcp\\dist\\index.js"]
  }
}
```

### 添加更多 MCP 服务器

在 `mcpServers` 下添加更多条目即可，OpenCode 支持同时运行多个 MCP 服务器。

---

## 📄 License

MIT — 随便用，引用请注明来源。
