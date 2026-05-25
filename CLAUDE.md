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

### 2.10 改前必读、写前必想（rule 08 · 物理强制）

> v0.11 新增。它把 rule 04（完整阅读）+ rule 02（七问）的最低必答子集**折叠为修改前置硬纪律**，并由 hooks 物理强制。

- **改前必读** — 任何 `Edit` 之前，本会话**必须**：完整 Read 目标文件 + Read 所有调用点上下文（≥ 20 行）+ Read 所有同步连带文件（修 `rules/*.md` → 读 `prompts/` + `commands/` + `docs/RULES.md`；修 hook → 读 `hooks/hooks.json` + 本 doc §8 + 对应 `tests/test_*.py`）。
- **写前必想** — 任何 `Edit` / `Write` 之前，必须在思维链或最终回复中**显式回答至少 3 项**：根因 / 架构定位 / 方案触底 / 连带影响 / 风险 / 方案对比。
- **物理强制**：
  - `PreToolUse(Edit|Write)`：未读已存在文件 → DENY（v0.3.2 read_guard，rule 04 + rule 08）
  - `Stop` **layer (e)**：本轮做了 Edit 但回复缺"系统式自答"标记（< 3 个 rule-02 关键词 且无 rule 08 标记）→ BLOCK（v0.11）
- 详见 [`rules/08-read-before-edit-think-before-write.md`](rules/08-read-before-edit-think-before-write.md)。

### 2.11 系统式修改，禁止打补丁（rule 09 · 物理强制）

> v0.11 新增；v0.13 加入 rolling-patch 频率层。它把 rule 03（修根因）的"反偷懒"清单**结构化为修改通用纪律**，并在 PreToolUse new_string 内容层 + PreToolUse 频率层 + Stop 收尾层物理强制。

- **修改必须系统性 + 完整性**，不允许局部打补丁。
- **打补丁式被禁**：局部 `try/except: pass` / 无 why 的 `# noqa` / `@ts-ignore` / `eslint-disable` / 在调用点包 wrapper 让异常消失但根因不动 / `time.sleep` 掩盖竞态 / 把测试断言放宽 / 把 timeout 拉长 / 注释掉失败测试 / 留 TODO 当作"完成" / **rolling patches（同一文件本会话 ≥ 4 次小幅 Edit）**。
- **每处屏蔽标记必带 why 注释**才允许（含 `because` / `原因` / `why` / 显式说明）。
- **物理强制**：
  - `PreToolUse(Edit|Write)` **内容层**：new_string 含未带 why 注释的 `try/except: pass` / `# noqa` / `# type: ignore` / `// @ts-ignore` / `// @ts-expect-error` / `// eslint-disable` / `time.sleep(...) # race/wait/workaround` → DENY（v0.11 patch-style detector）
  - `PreToolUse(Edit|Write)` **频率层（v0.13）**：同一文件本会话第 4 次小幅 Edit（≤ 10 行 且 < 200 字符）且**没有**一次系统式重写（≥ 50 行 / ≥ 1500 字符）介入 → DENY；DENY 时不增计数器，后续小 Edit 持续被拦，直到一次系统式 Edit/Write 把计数器清 0。
  - `PreToolUse(Bash)`：`--no-verify` / `--no-gpg-sign` / `git push --force` / `chmod 777` → DENY（v0.3 bash_guard）
  - `Stop` **layer (f)**：本轮做了 Edit 但回复缺"根因 + 影响 + 方案"三件套标记 → BLOCK（v0.11）
- 详见 [`rules/09-systematic-modification.md`](rules/09-systematic-modification.md)。

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
│   ├── RULES.md                     # 规则目录索引
│   └── EDICTS.md                    # 圣旨使用指南（v0.12）
├── rules/                           # ★ LLM-agnostic 规则源文件（纯 Markdown）
│   ├── 00-index.md
│   ├── 01-verify-dont-guess.md
│   ├── 02-systematic-not-reactive.md
│   ├── 03-root-cause.md
│   ├── 04-full-context.md
│   ├── 05-cite-sources.md
│   ├── 06-verify-convergence.md
│   ├── 07-task-fidelity.md
│   ├── 08-read-before-edit-think-before-write.md  # v0.11
│   ├── 09-systematic-modification.md              # v0.11
│   └── en/                                        # 英文镜像（v0.6.2+）
├── prompts/                         # 给钩子注入用的提示词片段（v0.12 瘦身 54%）
│   ├── session-start.md             # SessionStart 注入（中文 canonical）
│   ├── user-prompt.md               # UserPromptSubmit 注入（中文 canonical）
│   └── en/                          # 英文镜像（v0.15；CC_ENSLAVER_LANG=en 切换）
│       ├── session-start.md
│       └── user-prompt.md
├── hooks/
│   ├── hooks.json                   # 钩子注册（Claude Code 适配层）
│   └── scripts/
│       ├── inject_context.py        # 软层：会话/每轮注入（含圣旨注入）
│       ├── read_guard.py            # PreToolUse(Read|Edit|Write) 守卫
│       ├── bash_guard.py            # PreToolUse(Bash) 守卫
│       ├── register_read.py         # Read-cache escape hatch (v0.4)
│       ├── stop_guard.py            # Stop 6 层决策（v0.12 表格化输出）
│       ├── gc_state.py              # 手动 GC (v0.6.1)
│       ├── manage_edicts.py         # 圣旨 CRUD CLI (v0.12)
│       └── lib/
│           ├── state.py             # 跨钩子持久状态
│           └── edicts.py            # 圣旨加载/注入/匹配 (v0.12)
├── commands/                        # 用户主动调用的 slash 命令
│   ├── checklist.md                 # /cc-enslaver:checklist
│   ├── verify.md                    # /cc-enslaver:verify
│   ├── gc.md                        # /cc-enslaver:gc
│   └── edict.md                     # /cc-enslaver:edict (v0.12)
├── agents/
│   └── verifier.md                  # 独立验证子代理
├── skills/
│   └── systematic-debug/
│       └── SKILL.md                 # debug 语境自动唤起
└── tests/                           # 135 个黑盒测试（v0.12 +31）
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
- 注入语言切换（v0.15）：默认中文；设置环境变量 `CC_ENSLAVER_LANG=en` 切换为英文注入（读 `prompts/en/*.md`），未知值 / 未设置回退中文

---

## 6. 当前版本

`v0.12.0` —— **圣旨（Imperial Edicts）+ Stop 输出表格化 + prompts 瘦身 54%**。

v0.12 一次性回答用户三个使用反馈：

1. **软层提醒强度不够** — `session-start.md` 219 → 89 行 + `user-prompt.md` 41 → 31 行（共瘦身 54%）。9 条规则改成单行表格、物理强制改成 4 行触发表、回复骨架改成 5 行阶段表，密度大幅提升以抵御 context 老化。
2. **Stop 收尾杂乱** — 6 层独立长说教（每层 ~50 行）改为**统一格式**：每个 block reason 必含 `cc-enslaver · Stop check FAILED at Layer (X) [rule NN]` 标题 + 6 行状态表（✅ Pass / ❌ FAIL / ⏸ pending / — n/a）+ 仅失败层的 5-10 行 Recovery + 一次性守卫脚注。`stop_guard.py` 通过 `LAYER_META` + `_render_status_table` + `_build_block_reason` 统一渲染；8 个新测试锁定格式契约。
3. **圣旨 / Imperial Edicts** —— 项目级用户自定义硬规则系统。
   - 文件：`${CLAUDE_PROJECT_DIR}/.claude/cc-enslaver/edicts.toml`（项目级，可入 git；fallback 到 `~/.claude/cc-enslaver/edicts.toml`）
   - 格式：TOML 数组 `[[edicts]]`，字段 `id` / `text` / `severity` (`must`|`should`) / `deny_edit` / `deny_bash` / `note`
   - 注入：`SessionStart` + `UserPromptSubmit` 都注入（每轮重注入抵御 context 老化）
   - 物理强制：`PreToolUse(Edit|Write)` 扫 `new_string`、`PreToolUse(Bash)` 扫 `command`，命中 must edict 即 DENY
   - 管理：`/cc-enslaver:edict list / add / remove / reload / path` slash command + `hooks/scripts/manage_edicts.py` CRUD CLI
   - 设计契约：内置 9 条规则**先跑**、圣旨**后跑**——圣旨不能用来 whitelist `--no-verify` 等内置绕过拦截
   - 文档：[`docs/EDICTS.md`](docs/EDICTS.md)

之前版本要点保留：

- **v0.11.0** —— rule 08 (改前必读 / 写前必想) + rule 09 (系统式修改 / 禁止打补丁) + Stop layer (e)+(f) + PreToolUse(Edit|Write) 补丁标记物理拦截。
- **v0.10.0** —— `systematic-debug` skill 加入 Step 0 = build feedback loop（10 种 loop 形态 + 4-question 检查）。
- **v0.9.1** —— Stop hook 修复 silent no-op bug（v0.6.0-v0.9.0 期间 layer (a-d) 实际未触发，根因是 transcript JSONL 字段路径错误 + 尾部 tool_use 覆写）。
- **v0.9.0** —— 项目重命名 `anti-laziness` → `cc-enslaver`。
- **v0.8.0** —— 新增 rule 07（任务忠实 / 请求覆盖 / 无降级）+ Stop hook Layer (d) 强制收尾自答。

- ✅ 标准 Claude Code 插件目录结构
- ✅ `rules/` **9** 条核心规则（中文 canonical + [`rules/en/`](rules/en/) 英文镜像，v0.6.2 / v0.8.0 / v0.11.0 同步）
- ✅ SessionStart / UserPromptSubmit 钩子注入（软层 + v0.11 加入"标准回答骨架"与"每轮硬性自检清单"）
- ✅ **PreToolUse(Read\|Edit\|Write) 统一处理**：Read 录入、Edit/Write 检查未读已存在文件 → DENY；Edit/Write 检查 new_string 补丁标记 → DENY（v0.11）；成功 Edit/Write 记录 `last_edit_turn`（v0.11）
- ✅ **PreToolUse(Bash) 绕过模式硬拦截**：`--no-verify` / `--no-gpg-sign` / `git push --force`（不含 `--force-with-lease`） / `chmod 777` 命中即 deny
- ✅ **Read-cache escape hatch**（v0.4.0）：`register_read.py` + `bash_guard` 重算 SHA-256 闸门
- ✅ **Stop 钩子六层决策**（v0.6.0 → v0.7.0 → v0.8.0 → v0.11.0 累加）：[`stop_guard.py`](hooks/scripts/stop_guard.py) 在每次 Stop 检查 agent 末尾消息：
  - **Layer (a) v0.6.0**：含 done-claim 但完全无 evidence → 拒（rule 06 base）
  - **Layer (b) v0.7.0**：含 done-claim + hedge 在 50 字符内 → 拒（rule 01 投影）
  - **Layer (c) v0.7.0**：含 done-claim + evidence 但缺收敛标记 且自答 4 题命中 < 2 → 拒（rule 06 deep）
  - **Layer (d) v0.8.0**：通过 (a)(b)(c) 但缺忠实标记 且自答 3 题命中 < 2 → 拒（rule 07）
  - **Layer (e) v0.11.0**：本轮 `last_edit_turn == turn_count` 且缺 rule-08 标记 且 rule-02 关键词命中 < 3 → 拒（rule 08）
  - **Layer (f) v0.11.0**：本轮做了 Edit 且缺 rule-09 标记 且"根因 + 影响 + 方案"三件套不全 → 拒（rule 09）
  - 一次性守卫：`last_blocked_turn` 持久化，turn ∈ `[last+1, last+3]` 宽限窗口内不重复 block
- ✅ 跨钩子持久状态：`${CLAUDE_PLUGIN_DATA}/sessions/<sid>.json`（`read_files` / `last_blocked_turn` / `last_edit_turn`；路径规范化、跨平台、failing-open）
- ✅ 4 个 slash 命令（`/cc-enslaver:checklist` + `/cc-enslaver:verify` + `/cc-enslaver:gc` + `/cc-enslaver:edict`（**v0.12**））
- ✅ 1 个 verifier 子代理
- ✅ 1 个 systematic-debug 自动唤起 skill（v0.10 加入 Step 0 = build feedback loop）
- ✅ `.claude-plugin/marketplace.json`：本地安装入口
- ✅ **测试套件** [`tests/`](tests/) **135 个**：v0.12 新增 `test_edicts.py`（23 个 — 加载 / 注入 / Edit/Write/Bash DENY / severity gating / 内置先跑）+ `TestV012StatusTableFormat`（8 个 — Stop 表格格式契约）
- ✅ **圣旨（v0.12）**：[`hooks/scripts/lib/edicts.py`](hooks/scripts/lib/edicts.py) + [`manage_edicts.py`](hooks/scripts/manage_edicts.py) + [`commands/edict.md`](commands/edict.md) + [`docs/EDICTS.md`](docs/EDICTS.md)
- ✅ **手动 GC**（v0.6.1）：[`hooks/scripts/gc_state.py`](hooks/scripts/gc_state.py) + [`commands/gc.md`](commands/gc.md)
- ✅ **GitHub Actions CI**（v0.5.1）：matrix `ubuntu-latest` × `windows-latest` × Python `3.13`

未实现（见 [`CHANGELOG.md`](CHANGELOG.md) 路线图）：

- ⏳ Stop 钩子的**深度文件声明验证**（解析"我修改了 X" → 验 git diff / mtime）
- ⏳ Auto-GC on SessionStart（v0.6.1 只做了手动 GC）
- ⏳ Rolling-patch PreToolUse 硬拦截（v0.11 仅 Stop layer (f) 软层兜底 + rule 09 doc 软纪律；硬拦截需 `edits_per_file` 计数器）
- ⏳ 英文 prompts（v0.6.2 / v0.11 翻了 rules/en/ 全部 9 条，但 prompts/ 仍是中文）
