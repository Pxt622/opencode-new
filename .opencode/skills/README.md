# OpenCode Skills Collection

本仓库收集了适用于 OpenCode 的所有 Skills，便于分享和管理。

## Skills 目录

本仓库的 `skills/` 目录下包含以下 Skills：

### OpenCode 原生 Skills
| Skill 名称 | 描述 |
|-----------|------|
| `openspec-apply-change` | 实现 OpenSpec 变更任务 |
| `openspec-archive-change` | 归档已完成的 OpenSpec 变更 |
| `openspec-explore` | 探索模式 - 思考伙伴 |
| `openspec-propose` | 提出新变更 - 一站式生成提案 |
| `skill-seekers` | 技能发现和管理 |
| `web-access` | 网页访问和抓取 |
| `automation-workflows` | 自动化工作流 |

### HarmonyOS 开发 Skills
| Skill 名称 | 描述 |
|-----------|------|
| `harmony-arkts` | HarmonyOS ArkTS/ArkUI 开发技能 |

### 设计与 UI/UX Skills
| Skill 名称 | 描述 |
|-----------|------|
| `algorithmic-art` | 算法艺术生成 |
| `banner-design` | Banner 设计 |
| `brand` | 品牌设计 |
| `brand-guidelines` | 品牌指南 |
| `canvas-design` | Canvas 设计 |
| `design` | 设计技能 |
| `design-system` | 设计系统 |
| `frontend-design` | 前端设计 |
| `slides` | 幻灯片设计 |
| `ui-styling` | UI 样式设计 |
| `ui-ux-pro-max` | UI/UX Pro Max |

### 文档与内容 Skills
| Skill 名称 | 描述 |
|-----------|------|
| `doc-coauthoring` | 文档协作 |
| `docx` | Word 文档处理 |
| `internal-comms` | 内部沟通 |
| `pdf` | PDF 处理 |

### 办公文档 Skills
| Skill 名称 | 描述 |
|-----------|------|
| `pptx` | PowerPoint 处理 |
| `xlsx` | Excel 处理 |

### 开发工具 Skills
| Skill 名称 | 描述 |
|-----------|------|
| `claude-api` | Claude API 使用 |
| `mcp-builder` | MCP 服务器构建 |
| `skill-creator` | Skill 创建工具 |
| `web-artifacts-builder` | Web 构建工具 |
| `webapp-testing` | Web 应用测试 |

### 创意 Skills
| Skill 名称 | 描述 |
|-----------|------|
| `slack-gif-creator` | Slack GIF 创建 |
| `theme-factory` | 主题工厂 |

## 安装

将 `skills/` 目录复制到你的 OpenCode 配置目录：

```bash
# 对于 Windows
cp -r skills/ "$env:USERPROFILE/.opencode/"

# 对于 macOS/Linux
cp -r skills/ ~/.opencode/
```

## 使用方法

在 OpenCode 中加载 Skill：

```
/skill <skill-name>
```

例如：
```
/skill harmony-arkts
/skill web-access
/skill design-system
```

## 贡献

欢迎提交新的 Skills！请确保：
1. 每个 Skill 包含 `SKILL.md` 文件
2. Skill 描述清晰明确
3. 包含必要的脚本和参考资料

## 许可证

各个 Skill 可能具有不同的许可证，请参考各自目录中的 `LICENSE.txt` 文件。
