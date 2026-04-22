# Superpowers 插件使用最佳实践

> 基于 [obra/superpowers](https://github.com/obra/superpowers) v5.0.7 的源码分析整理。
> 适用平台：Claude Code / OpenAI Codex / Cursor / Gemini CLI / Copilot CLI / OpenCode。

## 一、先理解它到底是个什么东西

Superpowers 不是一个"工具包"，它是**一套强制走流程的软件工程方法论**，用 skill 文件把流程写成"AI 必须遵守的契约"。

它解决的痛点是：AI 接到需求后的默认行为是"立刻开始写代码"，而这会把"需求不清"、"设计缺失"、"没有测试"、"完成了但其实没验证"这些问题全部甩给后面。Superpowers 把这个链路掰直成：

```
需求 → 设计(spec) → 计划(plan) → 隔离工作区 → TDD 实现 → 两阶段审查 → 验证 → 合并/PR
```

每一步都有一个对应的 skill 在把关,并且 skill 里写着 "Iron Law"(铁律)和 "Red Flags"(红旗词),用来堵 AI 自己给自己找的借口。

## 二、它怎么做到"自动生效"

搞懂这个机制非常重要,否则你会困惑"我怎么触发它"。

### 启动链路(读 `hooks/session-start` 的结论)

1. 你装了 superpowers 插件后,每次 Claude Code 会话启动、清屏、压缩上下文时,`SessionStart` 钩子都会跑。
2. 钩子直接把 `skills/using-superpowers/SKILL.md` 的全文塞进 `additionalContext`,用 `<EXTREMELY_IMPORTANT>` 标签包起来。
3. 这个 skill 的核心就一条:**"只要有 1% 的可能某个 skill 适用,就必须调用 Skill 工具"**。

所以你根本不用手动 `/xxx`(那几个 command 都标了 `Deprecated`)。你只要正常说话:

- 说"我想加个 X 功能" → 触发 `brainstorming`
- 说"这个 bug" / "测试挂了" → 触发 `systematic-debugging`
- 说"写完了" / "搞定了" → 触发 `verification-before-completion`

### 唯一你要记住的 skill 名字

| 场景 | Skill |
|---|---|
| 要做任何新东西 | `superpowers:brainstorming` |
| 有 spec、要拆任务 | `superpowers:writing-plans` |
| 有 plan、开干 | `superpowers:subagent-driven-development`(首选)或 `superpowers:executing-plans` |
| 写功能或修 bug | `superpowers:test-driven-development` |
| 碰到任何异常 | `superpowers:systematic-debugging` |
| 声称"完成了" | `superpowers:verification-before-completion` |
| 收尾(合并/PR) | `superpowers:finishing-a-development-branch` |

## 三、推荐的主干工作流(一条龙)

这是 README 里建议、且在 skill 之间用 "REQUIRED SUB-SKILL" 互相链着的一条路径。**单人开发照搬这一条**就够。

```
[你]"想做 X 功能"
   └─► brainstorming
        ├─ 先探项目上下文(读文件、commits)
        ├─ 一次只问一个问题,澄清意图/约束/成功标准
        ├─ 提 2-3 种方案 + 推荐
        ├─ 分节呈现设计,每节得到你点头
        ├─ 写进 docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md 并 commit
        └─ HARD-GATE:你没批 spec 之前,它不写一行代码

   └─► using-git-worktrees(自动被 brainstorming 调起)
        ├─ 优先 .worktrees/,检查是否在 .gitignore 里(不在就先加再提交)
        ├─ git worktree add .worktrees/<branch> -b feature/<x>
        └─ 跑一次测试,确认基线干净

   └─► writing-plans
        ├─ 拆到 2-5 分钟/步的粒度
        ├─ 每步写明:确切文件路径 + 完整代码块 + 确切命令 + 预期输出
        ├─ 零占位符(禁止 TBD、"添加适当的错误处理"这种话)
        └─ 存到 docs/superpowers/plans/YYYY-MM-DD-<feature>.md

   └─► subagent-driven-development(推荐)
        └─ 每个任务:
            ├─ 派 implementer 子 agent(用 TDD,红绿重构)
            ├─ 派 spec-reviewer 子 agent 审"做的是不是 spec 要求的"
            ├─ 派 code-quality-reviewer 子 agent 审"做得好不好"
            ├─ 有问题让 implementer 修,再审,直到双绿
            └─ 在 TodoWrite 里打勾

   └─► finishing-a-development-branch
        ├─ 先跑全量测试
        └─ 给你 4 选 1:本地 merge / 推 PR / 保留 / 丢弃
```

## 四、单条 skill 的几个关键原则

### `brainstorming` — 不许跳过设计

- **HARD-GATE**:就算只是改 config、加个小工具,也要先写一段(哪怕两三句)设计并让你批准。
- 一次问一题,尽量给选项。
- 发现需求太大、含多个独立子系统时,它会主动建议**先拆项目**。

### `test-driven-development` — Iron Law 很重

- `NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST`
- 如果它先写了实现代码再补测试,按 skill 的要求是**把实现删掉重写**。这点它会很轴,别劝它"保留作参考" —— skill 里专门把这个借口堵死了。
- 节奏:写测试 → 运行并看到它失败(且失败原因正确)→ 写最少代码让它过 → 再看它过 → 重构。

### `systematic-debugging` — 四阶段不能跳

1. 根因调查(读完错误、复现、查近期改动、加诊断日志)
2. 模式分析(找能工作的类似代码做对比)
3. 假设测试(单变量小改动)
4. 实现(先写失败测试再修)

特别要记:**连续 3 次修都没修好,要停下来质疑架构**,而不是试第 4 个修法。

### `verification-before-completion` — 不准吹牛

声称"完成/通过/搞定"之前,必须在**当前这条消息里**刚刚跑过对应验证命令并看到输出。"应该过了"、"看起来对了"是禁用语。

### `subagent-driven-development` — 吃子 agent 红利

- **每个任务都派一个全新子 agent**,不继承你的上下文,你只给它精确剪裁过的任务文本 + 必需上下文。
- 两阶段审查(spec 合规 → 代码质量)都用子 agent 来做,顺序不能反。
- 模型挑选:机械任务用便宜快的模型,集成/判断用标准模型,设计/审查用最强模型。

### `using-git-worktrees` — 每个 feature 一个隔离副本

- 优先使用 `.worktrees/`(隐藏),其次 `worktrees/`,都没有则读 CLAUDE.md 偏好,再没有才问你。
- **强制校验目录在 .gitignore 中**,否则先改 .gitignore 再建。

### `writing-skills` — 自己造 skill 也得 TDD

想给自己的工作空间造私有 skill 时:

1. 先用**无 skill 的子 agent**跑一遍场景,记录它自然出现的错误和"借口原文"。
2. 针对那些具体借口写 skill(不要泛化)。
3. 再跑场景看是否遵守。
4. 出现新借口 → 加明确反驳 → 再跑。

**关键坑**:`description` 字段只写"什么时候用",**绝不能写流程摘要** —— 否则 Claude 会照着摘要办而跳过读正文。

## 五、使用侧的最佳实践清单

针对你(user)怎么配合它,这些会让效果翻倍:

1. **不要用祈使句打断流程**。"直接给我写 X" 会让它把 brainstorming 判成可跳过。说"我想做 X"就好。
2. **尊重它的"一次一问"**。它不是笨,是在帮你 de-risk。
3. **spec 写完后你真的去读一遍**。skill 里专门设了 "User Review Gate" 让你把关,别直接说"ok 继续"。
4. **plan 里看到 TBD、"添加适当的错误处理"这种话立刻打回** —— skill 说了这是计划失败。
5. **子 agent 失败时别手动救场**。把反馈给主 agent,让它派一个新子 agent 修,避免你主会话的上下文被污染。
6. **复用你项目的 CLAUDE.md 来压过 skill 默认值**。比如在 CLAUDE.md 里写 "worktree directory: .worktrees",`using-git-worktrees` 就不会再问你。
7. **默认走 subagent-driven,而不是 executing-plans**。Claude Code 支持子 agent,效果明显更好(README 也特别说明了这一点)。
8. **skill 正在跑时它会宣布 "I'm using the X skill to Y."** —— 看到这句说明 skill 真的生效了;没有这句就是没在走流程。

## 六、容易踩的误区

- **以为 `/brainstorm`、`/write-plan`、`/execute-plan` 是主要入口**。这仨 command 在仓库里全是 Deprecated 的占位文件,内容就一句话"告诉用户去用对应 skill"。**正确入口是自然语言 + Skill 工具自动触发**。
- **把 Superpowers 当外接工具**。它不是调用 API,它是改你 Claude 的行为模式,本质是"系统提示工程"。
- **在 main 分支直接让它跑实现**。几个 skill 里都写了 "Never start implementation on main/master without explicit user consent"。你要么先 checkout feature 分支,要么让它用 worktree。
- **跟它 "You're absolutely right!"**。`receiving-code-review` skill 明确禁止这种表演式附和。它在同样逻辑下也不会对你这样说。

## 七、和 moon-ai-work 的结合建议

基于工作空间当前的结构(`moon-agent-os`、`moon-share`、`moon-skills`,都是 main + feature/* 策略,单人开发),建议:

1. **在每个子项目的 CLAUDE.md 里写一段 Superpowers 偏好**,省掉每次被问:

```md
## Superpowers preferences
- Worktree directory: .worktrees
- Spec location: docs/specs/YYYY-MM-DD-<topic>-design.md
- Plan location: docs/plans/YYYY-MM-DD-<feature>.md
- Branch naming: feature/<short-slug>
- Default execution: subagent-driven-development
```

2. 首次在一个子项目用它时,让它**故意挑个小需求**(比如 moon-share 新增一篇文章的脚手架)走完完整链路,你能直观看到每个 skill 在什么时机介入。

3. `moon-skills` 仓库很可能也想做类似的技能集合 —— 直接把 `skills/writing-skills/SKILL.md` 作为技能编写规范参考,但不要照搬那套 TDD 测试流程到琐碎 skill 上(它本身是给行为约束类 skill 设计的)。

## 附录:核心文件索引

从 superpowers 仓库结构反推的关键阅读顺序:

1. `skills/using-superpowers/SKILL.md` — 入口 meta skill
2. `skills/brainstorming/SKILL.md` — 设计阶段
3. `skills/writing-plans/SKILL.md` — 计划阶段
4. `skills/subagent-driven-development/SKILL.md` — 执行阶段(首选)
5. `skills/test-driven-development/SKILL.md` — 实现纪律
6. `skills/systematic-debugging/SKILL.md` — 调试纪律
7. `skills/verification-before-completion/SKILL.md` — 完成纪律
8. `hooks/session-start` — 自动注入机制
9. `skills/writing-skills/SKILL.md` — 如需自造 skill
