---
name: baoyu-url-to-markdown
description: 通过 Chrome CDP 获取任意 URL 并转换为 Markdown。支持两种模式 - 页面加载自动捕获，或等待用户信号（适用于需要登录的页面）。当用户想要将网页保存为 Markdown 时使用。
---

# URL 转 Markdown

通过 Chrome CDP 获取任意 URL 并将 HTML 转换为干净的 Markdown。

## 脚本目录

**重要**：所有脚本位于此 skill 的 `scripts/` 子目录中。

**Agent 执行指令**：
1. 确定此 SKILL.md 文件的目录路径为 `SKILL_DIR`
2. 脚本路径 = `${SKILL_DIR}/scripts/<script-name>.ts`

## 功能特性

- Chrome CDP 完整 JavaScript 渲染
- 两种捕获模式：自动或等待用户信号
- 干净的 Markdown 输出，包含元数据
- 通过等待模式处理需要登录的页面

## 使用方法

```bash
# 自动模式（默认）- 页面加载时捕获
npx -y bun ${SKILL_DIR}/scripts/main.ts <url>

# 等待模式 - 在捕获前等待用户信号
npx -y bun ${SKILL_DIR}/scripts/main.ts <url> --wait

# 保存到指定文件
npx -y bun ${SKILL_DIR}/scripts/main.ts <url> -o output.md
```

## 选项说明

| 选项 | 描述 |
|------|------|
| `<url>` | 要获取的 URL |
| `-o <path>` | 输出文件路径（默认：自动生成）|
| `--wait` | 在捕获前等待用户信号 |
| `--timeout <ms>` | 页面加载超时（默认：30000ms）|

## 捕获模式

| 模式 | 行为 | 适用场景 |
|------|------|----------|
| 自动（默认） | 网络空闲时捕获 | 公开页面、静态内容 |
| 等待（`--wait`） | 用户信号触发捕获 | 登录要求、懒加载、付费墙 |

**等待模式工作流程**：
1. 使用 `--wait` 运行 → 脚本输出 "Press Enter when ready"
2. 询问用户确认页面已就绪
3. 发送换行符触发捕获

## 输出格式

YAML front matter 包含 `url`、`title`、`description`、`author`、`published`、`captured_at` 字段，后面是转换后的 Markdown 内容。

## 输出目录

```
url-to-markdown/<domain>/<slug>.md
```
- `<slug>`：来自页面标题或 URL 路径（短横线命名，2-6个词）
- 冲突解决：追加时间戳 `<slug>-YYYYMMDD-HHMMSS.md`

## 环境变量

| 变量 | 描述 |
|------|------|
| `URL_CHROME_PATH` | 自定义 Chrome 可执行文件路径 |
| `URL_DATA_DIR` | 自定义数据目录 |
| `URL_CHROME_PROFILE_DIR` | 自定义 Chrome 配置目录 |

## 故障排除

- Chrome 未找到 → 设置 `URL_CHROME_PATH`
- 超时 → 增加 `--timeout` 值
- 复杂页面 → 尝试 `--wait` 模式

## 扩展支持

可通过 EXTEND.md 自定义配置。支持：默认输出目录 | 默认捕获模式 | 超时设置