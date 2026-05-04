# CLAUDE.md — cc-enslaver 项目说明

> 本文件是仓库内的 **项目级指令**，会在 Claude Code 每次启动该仓库时被自动加载。
> 它同时是这个插件存在的**第一性原则声明**：插件本身就是为了把这些原则强制注入到每一次 AI 协作中。

---

## 1. 项目目标

`cc-enslaver` 是一个 **Claude Code 插件 + LLM-agnostic 规则包**，目的只有一个：

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

### 2.8 改完必须收敛验证

- 任何修复 / 更新 / 补丁完成后，**禁止**直接声称完成；先做 [`rules/06-verify-convergence.md`](rules/06-verify-convergence.md) 的 5 个子步骤：
  1. **重触发原症状** — 用用户最初描述失败的同一条命令重跑，确认报错消失
  2. **边界 + 反向用例** — 至少 1 个边界 + 1 个反向用例
  3. **连带不破坏** — 既有测试 / lint / 类型检查全跑
  4. **强制自答 4 题** — 是否真解决？有无更好方案？哪些没验？验证是否合理？
  5. **量化优于定性**（性能 / 竞态 / 兼容性场景）— 数字 / 重跑 N 次 / 测试矩阵
- 上述任意一步揭示 "未解决 / 未覆盖" → **回到 2.1 七问**重新分析根源，再修，再验，**直到收敛**。
- 禁止：**"改完没报错"** / **"测试通过"** / **"本地能跑"** / **"看起来对了"** / **"上次类似的应该差不多"** 当作收敛证据。

### 2.9 声称完成前必须做忠实自答（rule 07）

- rule 06 验"修的部分对不对"（技术轴）；rule 07 验"用户**要求的全部**做了吗、按**原标准**做了吗"（契约轴）。两轴不同维度，**必须分别回答**，不允许互相替代。
- 任务声称完成前，**禁止**直接 ship；先做 [`rules/07-task-fidelity.md`](rules/07-task-fidelity.md) 的 5 步检查 + 3 题自答：
  1. **拆解原始请求** — 回到用户最初消息，列出所有显式动词命令、明示的程度词、隐含连带项
  2. **逐项核对** — 每个子项标 ✅ 完成 / ⚠️ 降级 / ❌ 未做 + 证据 / 原因
  3. **标准达到性** — 用户每个程度词（强制 / 完整 / 严格 / 所有 / 立即 / 全面）都落地为可验证的硬动作（钩子 / 断言 / 测试），不是软文档
  4. **范围不溢出** — 没做用户没要求的重构 / 抽象 / 改名
  5. **半成品声明** — 所有 TODO / FIXME / "暂时" 显式列出
  6. **自答 3 题** —
     - **覆盖性**：原始请求拆几项？做了哪几项？哪些没做？为什么？
     - **标准性**：每个程度词都落地为硬动作了吗？哪些只是软文档？
     - **忠实性**：偷换概念？降级标准？范围溢出？藏 TODO？
- 上述任意一项揭示 "遗漏 / 降级 / 偷换 / 半成品藏匿" → **回到检查 1** 或主动停下来跟用户对齐，**禁止**单方面宣告完成。
- 禁止：**"主要部分都做了"** / **"应该覆盖了"** / 用户要 A、B、C 但只交付 A、B / 用户要"强制"但实现是"软建议" / 留 TODO 说"完成" / 用 rule 06 的收敛报告代替 rule 07 自答。

---

## 3. 仓库结构（开发者视角）

> 完整架构图与组件职责见 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)。

```
cc-enslaver/
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
│   ├── 05-cite-sources.md
│   ├── 06-verify-convergence.md
│   └── 07-task-fidelity.md
├── prompts/                         # 给钩子注入用的提示词片段（汇总自 rules/）
│   ├── session-start.md             # SessionStart 注入内容
│   └── user-prompt.md               # UserPromptSubmit 注入内容
├── hooks/
│   ├── hooks.json                   # 钩子注册（Claude Code 适配层）
│   └── scripts/
│       └── inject_context.py        # 单一脚本，按 --event 切分注入逻辑
├── commands/                        # 用户主动调用的 slash 命令
│   ├── checklist.md                 # /cc-enslaver:checklist
│   └── verify.md                    # /cc-enslaver:verify
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

`v0.9.0` —— **项目重命名：`anti-laziness` → `cc-enslaver`（marketplace `agent-rigor` → `cc-enslaver`）**。所有五层名字（plugin name、marketplace name、GitHub repo、slash 命令前缀、状态目录基名）现在统一为单一标识符 `cc-enslaver`。无任何规则 / 钩子 / 测试行为变化，只是字面替换 + 版本号 bump。详见 `CHANGELOG.md` v0.9.0 段（含改名后果与连带影响）。

之前版本要点保留：

- **v0.8.0** —— 新增 rule 07（任务忠实 / 请求覆盖 / 无降级）+ Stop hook Layer (d) 强制收尾自答。rule 06 解决"症状-根因"轴（修的部分对不对），rule 07 解决"请求-交付"轴（用户要的全做了吗、按原标准做了吗）。两轴不同维度，**必须分别回答**。Layer (d) 在 (a)(b)(c) 全过后再检查：含 done-claim + evidence + rule-06 自答，**且**有 rule-07 标记（`rule 07` / `任务忠实` / `请求覆盖` / `无降级` / `task fidelity` / `no degradation` / ✅ 完成 列表行等）**或**至少 2/3 自答题（覆盖性 / 标准性 / 忠实性）匹配，才允许 Stop。

- ✅ 标准 Claude Code 插件目录结构
- ✅ `rules/` **7** 条核心规则（中文 canonical + v0.6.2 / v0.8.0 同步的 [`rules/en/`](rules/en/) 英文镜像）
- ✅ SessionStart / UserPromptSubmit 钩子注入（软层）
- ✅ **PreToolUse(Read\|Edit\|Write) 统一处理**（v0.3.2）：Read 录入会话状态、Edit/Write 检查未读已存在文件 → deny
- ✅ **PreToolUse(Bash) 绕过模式硬拦截**：`--no-verify` / `--no-gpg-sign` / `git push --force`（不含 `--force-with-lease`） / `chmod 777` 命中即 deny
- ✅ **Read-cache escape hatch**（v0.4.0）：`register_read.py` + `bash_guard` 重算 SHA-256 闸门
- ✅ **Stop 钩子四层决策**（v0.6.0 → v0.7.0 → v0.8.0 累加）：[`stop_guard.py`](hooks/scripts/stop_guard.py) 在每次 Stop 检查 agent 末尾消息：
  - **Layer (a) v0.6.0**：含 done-claim 但完全无 evidence → 拒（rule 06 base）
  - **Layer (b) v0.7.0**：含 done-claim + hedge 在 50 字符内（`我觉得` / `I think` / `probably` 等首人称不确定语）→ 拒（rule 01 投影）
  - **Layer (c) v0.7.0**：含 done-claim + evidence 但缺收敛标记（`rule 06` / `自答` / `收敛` / `重触发` 等）**且** 4 题（真解决 / 更好方案 / 哪些没验 / 验证合理）匹配 < 2 → 拒（rule 06 deep）
  - **Layer (d) v0.8.0**：通过 (a)(b)(c) 但缺忠实标记（`rule 07` / `任务忠实` / `请求覆盖` / `原始请求` / `无降级` / `无遗漏` / `task fidelity` / `request coverage` / `no degradation` / `no omission` / `no scope creep` / `covered all` / `all requested` / ✅ 完成 列表行）**且** 3 题（覆盖性 / 标准性 / 忠实性）匹配 < 2 → 拒（rule 07）
  - 一次性守卫：`last_blocked_turn` 持久化，turn ∈ `[last+1, last+3]` 宽限窗口内不重复 block
- ✅ 跨钩子持久状态：`${CLAUDE_PLUGIN_DATA}/sessions/<sid>.json`（路径规范化、跨平台、failing-open）
- ✅ 2 个 slash 命令
- ✅ 1 个 verifier 子代理
- ✅ 1 个 systematic-debug 自动唤起 skill
- ✅ `.claude-plugin/marketplace.json`：本地安装入口
- ✅ **测试套件** [`tests/`](tests/)：76 个 unittest（v0.6.0 +16 stop_guard / v0.6.1 +9 gc_state / v0.7.0 +7 hedge & quiz / v0.8.0 +7 fidelity layer + 2 inject_context）
- ✅ **手动 GC**（v0.6.1 新增）：[`hooks/scripts/gc_state.py`](hooks/scripts/gc_state.py) + [`commands/gc.md`](commands/gc.md) slash 命令；`--dry-run`/`--apply` 互斥；默认 30 天阈值；只动 `${CLAUDE_PLUGIN_DATA}/sessions/`
- ✅ **GitHub Actions CI**（v0.5.1）：matrix `ubuntu-latest` × `windows-latest` × Python `3.13`

未实现（见 [`CHANGELOG.md`](CHANGELOG.md) 路线图）：

- ⏳ Stop 钩子的**深度文件声明验证**（解析"我修改了 X" → 验 git diff / mtime；v0.7.0 加深 self-quiz、v0.8.0 加 fidelity 都是 message-side 启发式；**文件声明验真**仍是 v0.9 候选）
- ⏳ Auto-GC on SessionStart（v0.6.1 只做了手动 GC；自动需要 last_gc.txt 节流）
- ⏳ 英文 prompts（v0.6.2 翻了 rules/en/、v0.8.0 同步了 rules/en/07-* 但 prompts/session-start.md 仍是中文 → 仅 Claude Code 中文用户受益于钩子注入；英文镜像主要用于复制到其他 LLM）
