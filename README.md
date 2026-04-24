# moon-skills

Claude Code / Codex / Cursor 等 AI 编程 agent 的技能与工作流集合。

> 🛠 **Build in Public** · 这是 [moon-agent-os](https://github.com/LQY-920/moon-agent-os) 项目的配套技能仓库——单人 + AI 杠杆工作流的公开沉淀。

## 安装

### 通过 skills CLI（推荐）

安装全部 skill 到用户级：

```bash
npx skills add https://github.com/LQY-920/moon-skills -g -y
```

只装某一个 skill：

```bash
npx skills add https://github.com/LQY-920/moon-skills --skill moon-share-or-skill -g -y
```

`-g` 全局安装到用户级，`-y` 跳过所有交互确认。

### 手动安装

```bash
git clone https://github.com/LQY-920/moon-skills.git
cp -r moon-skills/skills/* ~/.claude/skills/
```

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

## 使用方式

安装后在 Claude Code 中直接调用斜杠命令：

```
/moon-share-or-skill 帮我判断刚完成的 S1.1 身份子系统适不适合做成文章
```

或在自然对话中提到触发词（"沉淀一下"、"总结经验"、"这个能抽成 skill 吗"），Claude 会自动加载对应 skill。

## 新增 skill 规范

写一个新 skill 时遵循 Superpowers 的 [writing-skills](https://github.com/obra/superpowers) 规范:

1. 目录位置:`skills/<skill-name>/SKILL.md`
2. 命名:小写连字符,不要驼峰、中文、空格
3. frontmatter 的 `description` 字段**只写"何时使用"**,不写流程摘要(否则 Claude 会按摘要跳过正文)
4. 正文用小标题组织,包含:何时触发、流程步骤、判断准则、反模式、输出纪律

## License

MIT
