# moon-skills

Claude Code / Codex / Cursor 等 AI 编程 agent 的技能与工作流集合。

## 目录结构

```
moon-skills/
├── skills/                          # 自定义 skill 目录
│   └── moon-share-or-skill/
│       └── SKILL.md
├── superpowers-best-practices.md    # Superpowers 插件使用指南
└── README.md
```

## Skill 清单

| Skill | 用途 |
|---|---|
| [moon-share-or-skill](skills/moon-share-or-skill/SKILL.md) | 把刚做完的工作经验沉淀到 moon-share(对外内容)或 moon-skills(对内能力),或判定不沉淀 |

## 新增 skill 规范

写一个新 skill 时遵循 Superpowers 的 [writing-skills](https://github.com/obra/superpowers) 规范:

1. 目录位置:`skills/<skill-name>/SKILL.md`
2. 命名:小写连字符,不要驼峰、中文、空格
3. frontmatter 的 `description` 字段**只写"何时使用"**,不写流程摘要(否则 Claude 会按摘要跳过正文)
4. 正文用小标题组织,包含:何时触发、流程步骤、判断准则、反模式、输出纪律
