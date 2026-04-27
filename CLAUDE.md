# CLAUDE.md — anti-laziness 项目说明

> 本文件是仓库内的 **项目级指令**，会在 Claude Code 每次启动该仓库时被自动加载。
> 它同时是这个插件存在的**第一性原则声明**：插件本身就是为了把这些原则强制注入到每一次 AI 协作中。

---

## 1. 项目目标

`anti-laziness` 是一个 **Claude Code 插件 + LLM-agnostic 规则包**，目的只有一个：

> **杜绝 Claude Code（或任何其他 AI 代码助手）的"偷懒行为"。**

"偷懒"在本项目中有明确定义（详见 [`rules/`](rules/) 目录），核心特征是：

- **反应式修改**而非系统式修改（只补丁表面，不理解架构）
- **猜测**代替验证（"我记得"、"应该是"、"可能"）
- **关键词检索依赖**代替完整阅读（grep 一下就改，不读上下文）
- **记忆依赖**代替核对当前状态（凭印象，不重新读源文件）
- **撒谎/未经证实的陈述**（说改了其实没改、说存在其实不存在）
- **根因绕过**（用 sleep 掩盖竞态、用 `--no-verify` 跳过钩子、吞掉异常）
- **任务半成品**（写了一半、留 TODO、只做表面）

---

## 2. 设计原则（开发本仓库时也必须遵守）

这些原则同时是本仓库代码/文档的开发规范，**也是插件向被它管控的 AI 注入的内容**。

### 2.1 系统式 ≫ 反应式

修改任何代码或文档前，必须先回答：

1. 当前完整架构是什么样？
2. 待修改的部分位于架构的哪个区域？
3. 该部分的职责是什么？上游/下游是谁？
4. 问题/需求的**根源**是什么？机理是什么？
5. 我的修改方案是否真的从底层解决问题，而不是掩盖症状？
6. 这个修改对架构的连带影响是什么？哪些下游需要同步调整？
7. 修改完成后，全局视角下是否真的解决了问题？

> **反面例子（禁止）**：bug 长什么样？后果是什么？改完没现象就行 → 这是反应式思维。

### 2.2 验证 ≫ 猜测

- 任何关于文件、API、符号、版本、报错信息、引用文献的断言，**必须当场读取/运行/grep 验证**。
- "我不知道"永远优于"自信地错"。无法验证就明说，并指出补足验证所需的信息。
- 引用必须给出 `file:line`、命令输出、章节/页码、commit hash —— **不要写"我记得…"、"我相信…"**。

### 2.3 完整阅读 ≫ 关键词检索

- 编辑/编写新内容前，**必须真实探索整个相关架构、完整阅读所有相关文件**。
- 关键词检索（grep）只能用于**定位**，不能用于**理解**。定位之后必须读上下文。
- 记忆可能过时；当前文件状态才是权威源。

### 2.4 根因 ≫ 症状

- 报错就找根因，不要 try/except 静默吞掉。
- 测试不过就修测试或修代码，不要 `--no-verify` 跳过。
- 竞态就修同步，不要加 `sleep`。
- 钩子失败就修钩子失败的根源，不要绕过。

### 2.5 引用 ≫ 模糊指代

- 提到代码位置时使用 `path/to/file.ext:LINE` 或 `path/to/file.ext:LINE_START-LINE_END` 格式。
- VS Code 扩展环境下使用 markdown 链接 `[file.ext:42](path/to/file.ext#L42)`。

### 2.6 最小有效更改

- 不做"顺手"重构，不做投机性抽象，不为想象中的未来需求设计。
- 三行重复优于过早抽象。
- 不写无信息量的注释（不解释 WHAT，只在 WHY 非显然时解释 WHY）。

### 2.7 全局更新记忆

- 更新记忆时必须**全局检查**：是否有错误、过时、冗余的条目。
- 记忆是过去某时刻的快照，使用前要核对当前状态。

---

## 3. 仓库结构（开发者视角）

> 完整架构图与组件职责见 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)。

```
anti-laziness/
├── .claude-plugin/
│   └── plugin.json                  # 插件清单（Claude Code 适配层）
├── CLAUDE.md                        # 本文件 —— 项目指令
├── README.md                        # 用户向：安装、使用、原理
├── LICENSE                          # MIT
├── CHANGELOG.md                     # 变更日志
├── .gitignore
├── docs/
│   ├── ARCHITECTURE.md              # 架构说明（开发者向）
│   └── RULES.md                     # 规则目录索引（详细解释每条规则）
├── rules/                           # ★ LLM-agnostic 规则源文件（纯 Markdown）
│   ├── 00-index.md
│   ├── 01-verify-dont-guess.md
│   ├── 02-systematic-not-reactive.md
│   ├── 03-root-cause.md
│   ├── 04-full-context.md
│   └── 05-cite-sources.md
├── prompts/                         # 给钩子注入用的提示词片段（汇总自 rules/）
│   ├── session-start.md             # SessionStart 注入内容
│   └── user-prompt.md               # UserPromptSubmit 注入内容
├── hooks/
│   ├── hooks.json                   # 钩子注册（Claude Code 适配层）
│   └── scripts/
│       └── inject_context.py        # 单一脚本，按 --event 切分注入逻辑
├── commands/                        # 用户主动调用的 slash 命令
│   ├── checklist.md                 # /anti-laziness:checklist
│   └── verify.md                    # /anti-laziness:verify
├── agents/                          # 子代理
│   └── verifier.md                  # 独立验证子代理
└── skills/
    └── systematic-debug/
        └── SKILL.md                 # 在 debug/修 bug 语境下自动唤起
```

**核心分工（"为什么这样切"）：**

| 层 | 内容 | 谁会读它 |
|---|---|---|
| `rules/` | 规则的**纯文本定义**，不依赖任何运行时 | 任意 LLM、人类开发者、被注入的 agent |
| `prompts/` | 从 `rules/` 浓缩出来的**注入文案** | 钩子脚本 |
| `hooks/`、`commands/`、`agents/`、`skills/` | Claude Code 的**适配层**：把 `prompts/` 与 `rules/` 接到 Claude Code 的运行时 | Claude Code |
| `.claude-plugin/plugin.json` | 让 Claude Code 识别本仓库为插件 | Claude Code 的插件加载器 |

> 这种切分让规则本身可以被**移植到任何 LLM**：直接把 `rules/` 当作 system prompt 片段加载即可。

---

## 4. 修改本仓库时的强制流程

> 这是元规则：**用本插件的规则来开发本插件本身**。

修改任何文件前：

1. **读完整文件** —— 不只看 diff 上下文。
2. **看连带文件** —— 例如改 `rules/01-verify-dont-guess.md`，必须同步检查：
   - `prompts/session-start.md` 是否引用了它
   - `prompts/user-prompt.md` 是否引用了它
   - `docs/RULES.md` 是否描述了它
   - `commands/checklist.md` 是否汇总了它
3. **思考全局影响** —— 修改后整体语义是否仍然一致？是否产生冗余/矛盾？
4. **验证产出** —— Claude Code 是否仍能正常加载？hooks JSON 是否有效？plugin.json 是否合规？

---

## 5. 元数据

- 当前用户：skymanbp（`skyman.bp@gmail.com`）
- 用户主语言：中文
- 平台：Windows 11 + VS Code + Claude Code 扩展 + Git Bash
- Python：`C:/Users/skyma/AppData/Local/Programs/Python/Python313/python.exe`（也在 PATH 上）
- 规则源语言：中文（与用户主语言一致；技术文档/代码注释为英文）

---

## 6. 当前版本

`v0.1.0` —— 初始骨架。已实现：

- ✅ 标准 Claude Code 插件目录结构
- ✅ `rules/` 5 条核心规则（中文）
- ✅ SessionStart / UserPromptSubmit 钩子注入
- ✅ 2 个 slash 命令
- ✅ 1 个 verifier 子代理
- ✅ 1 个 systematic-debug 自动唤起 skill

未实现（见 [`CHANGELOG.md`](CHANGELOG.md) 路线图）：

- ⏳ PreToolUse 硬性拦截（如：`Edit` 前未 `Read` 该文件则阻断）
- ⏳ Stop 钩子的状态化检查（避免无状态死循环）
- ⏳ 跨会话的 verification trace 持久化
