# SKILL使用方法与规范

`SKILL.md` 它本质上是一份写给 AI 的、可复用的**标准作业程序（SOP）**，把原本需要每次重复交代的操作流程固化成文件，让 AI 按确定性的步骤执行，而不是每次“自由发挥”。

---

## 什么时候应该写 `SKILL.md`？

**当你发现自己不断把同样的说明、检查清单或多步骤程序粘贴到对话中时，或者当 `AGENTS.md` 中的某一部分已经演变成一个“程序”而不是“事实”时，就应该把它抽出来，写成一个 `SKILL.md`。**

### 场景一：同一套操作流程被反复执行

你发现自己每次让 Agent 做“代码审查”、“发版检查”、“新增一个 CRUD 功能”时，都要重新解释一遍步骤、顺序和注意事项。这时候，这套流程就应该从临时 Prompt 变成 `SKILL.md`。

### 场景二：`AGENTS.md` 里的规则“膨胀”成了步骤

`AGENTS.md` 适合放**每次会话都必须遵守的短规则**（比如“目录架构以 `architecture.md` 为准”）。但如果你在里面写了“第一步检查 X，第二步运行 Y，如果 Z 则执行 W”这样的多步骤程序，它就已经不属于 `AGENTS.md` 的职责了。把它移到 `SKILL.md`，`AGENTS.md` 只保留一句“涉及发版时，加载 `deploy` skill”。

### 场景三：一类任务有确定的“最佳路径”，不希望 AI 自由发挥

比如“新增一个页面”这件事，在你项目里有固定的最佳顺序：先在 `contracts.md` 定义类型 → 在 `architecture.md` 规定的位置建文件 → 遵循 `conventions.md` 的命名 → 最后对照 `checklist.md` 自查。这套顺序写在任何一个现有文档里都不合适，因为它是一个**跨文档的执行程序**，`SKILL.md` 就是它的归属地。

### 场景四：需要打包脚本或模板一起复用

`SKILL.md` 所在的文件夹可以包含 `scripts/`、`templates/`、`references/` 等辅助文件。如果你有一个固定的脚手架脚本、报告模板或长参考文档，想让 Agent 在特定任务中自动调用，`SKILL.md` 是合适的封装容器。

**一个简单的判断口诀：问“怎么做”且步骤固定、需要反复复用 → 写 `SKILL.md`；问“是什么”且需要作为事实长期查阅 → 写进现有 docs 文档。**

---

## SKILL文档优先级

> 在 DSH（DeepSeek Harness）里，`SKILL.md` 的存放规则很明确：**它不是靠命令“安装”的，而是把文件放进 DSH 会自动扫描的特定“技能根目录”中，它就能被发现**。
> 核心原则是：**项目级技能随项目走，放在项目里；个人通用技能放用户级目录，全局可用。**

### 项目级技能：随代码库共享（推荐）

这是最常用的方式，适合与当前项目强相关的 SOP。把技能放在项目根目录下，它会随 Git 一起提交，团队成员拉取代码后即可直接使用。

**首选路径：`<项目根>/.dsh/skills/<skill-name>/SKILL.md`**

这个目录的发现优先级最高（rank 100），并且支持热重载，修改后通常无需重启客户端即可生效。一个完整的存放示例是：

```text
your-project/
├── .dsh/
│   └── skills/
│       └── crud-feature/          # skill 名称
│           └── SKILL.md           # 技能文件本身
├── src/
└── ...
```

**备选路径：`<项目根>/.agents/skills/<skill-name>/SKILL.md`**

如果你的项目里同时使用了其他 AI 编码工具，希望技能能在多个 Agent 工具间共享，可以放在这里。它的优先级（rank 200）排在 `.dsh/skills` 之后。

---

### 用户级技能：跨项目全局可用

如果某个技能（比如通用的“代码审查流程”、“周报生成”）适用于你所有的项目，就放在用户目录下。

- **DSH 专用用户级路径**：`~/.dsh/skills/<skill-name>/SKILL.md`（rank 400）。Windows 下对应 `C:\Users\<用户名>\.dsh\skills\`。
- **跨 Agent 共享用户级路径**：`~/.agents/skills/<skill-name>/SKILL.md`（rank 500）。一些工具甚至将此处视为技能的“canonical 真身”。

一个安装示例是：

```bash
# macOS / Linux
mkdir -p ~/.dsh/skills/my-skill
cp SKILL.md ~/.dsh/skills/my-skill/SKILL.md

# Windows (PowerShell)
New-Item -ItemType Directory -Force "$HOME\.dsh\skills\my-skill"
Copy-Item SKILL.md "$HOME\.dsh\skills\my-skill\SKILL.md"
```

---

### DSH 的发现顺序与优先级

当多个位置存在同名技能时，DSH 按“rank 值越小越优先”的规则处理。以下是完整的扫描根列表，你只需记住常用的是前两个：

|     Rank     | 来源                           | 路径                        | 典型用途                     |
| :-----------: | :----------------------------- | :-------------------------- | :--------------------------- |
| **100** | **项目级 (`.dsh`)**    | `<项目根>/.dsh/skills`    | **首选**，项目专用 SOP |
| **200** | **项目级 (`.agents`)** | `<项目根>/.agents/skills` | 项目内多 Agent 工具共享      |
|      300      | 自定义配置                     | `Config.customSkillDirs`  | 特殊目录映射                 |
|      400      | 用户级 (`.dsh`)              | `~/.dsh/skills`           | 个人全局技能                 |
|      500      | 用户级 (`.agents`)           | `~/.agents/skills`        | 跨 Agent 全局共享            |
|      600      | 随包目录                       | `Config.bundledSkillDir`  | DSH 内置技能                 |

---

### 目录结构要求与注意事项

1. **标准的目录结构**：DSH 的技能发现器默认按 `<root>/<name>/SKILL.md` 的**一层深度**来扫描。因此，`SKILL.md` 必须放在一个**以技能名命名的独立文件夹**内，不能直接散落在 `skills/` 根目录下。
2. **扁平文件格式**：（不推荐）除了目录结构，DSH 也支持将技能写成一个独立的 `.md` 文件直接放在扫描根下，即 `<name>.md` 格式。
3. **项目根的判定**：`<项目根>` 指的是包含 `.git` 目录的最近祖先目录；如果项目没有初始化 Git，则使用当前工作目录（cwd）作为根。
4. **项目级优先于用户级**：如果同一个技能名在项目里和用户目录下都存在，DSH 会优先使用项目内的版本。因此，你可以用项目级技能来覆盖或“锁定”某个通用技能在特定项目下的行为。
5. **不要手动管理“真身”**：如果你使用技能管理类插件，它会将 `~/.agents/skills` 作为 canonical 位置来写入和锁定。手动在这个目录里新增或修改文件，可能与管理逻辑冲突，建议通过插件的安装/导入功能来管理全局技能。

**简单决策：跟着项目走的 SOP，放在项目根的 `.dsh/skills/` 下；你个人跨项目使用的通用流程，放在 `~/.dsh/skills/` 下。**

## 文件命名

`SKILL.md`

- 要求“SKILL”必须大写
- 虽然DSH支持用技能名称直接命名（扁平化结构），但是非常不推荐。

## 怎么写 `SKILL.md`？

### 结构

YAML frontmatter + Markdown 正文

### `SKILL.md` 由两部分组成

**第一部分：YAML frontmatter（元数据，决定“什么时候用我”）**:

```yaml
---
name: crud-feature
description: >
  当用户要求新增或修改一个 Todo 相关功能时，主动加载此技能。
  它规定了从契约定义到验收自查的完整标准流程。
---
```

关键点在于 `description`：**它是 Agent 决定是否调用该技能的唯一依据**，必须写清楚“做什么、什么时候用、核心价值是什么”。写模糊了，Agent 就匹配不准；写清楚了，触发才精准。

**第二部分：Markdown 正文（执行程序，决定“怎么干”）**:

正文是具体的操作流程，建议包含以下章节：

```markdown
## 概述
说明这个 skill 解决什么问题、适用边界。

## 前置条件
执行前必须满足的条件或必须先读的文档（比如“先读 contracts.md 确认类型”）。

## 工作流程
按顺序列出确定性步骤，每一步都明确“做什么、参考哪份文档、输出什么”。

## 异常处理
遇到什么情况该怎么应对（比如“类型定义冲突时，停下来引用 decisions.md 记录决策”）。

## 示例（可选）
一个完整的输入→输出示例，帮助 Agent 理解期望格式。
```

**正文的写作原则**：**模块化与单一职责**。一个 `SKILL.md` 只做一类事。与其写一个“通用开发 skill”，不如拆成 `crud-feature`、`code-review`、`release-checklist` 等独立 skill，每个的 `description` 才能精准匹配，指令也不会过长导致浪费上下文。

---

## 项目级技能存档结构

### 存档结构：目录形式（推荐）

```text
项目根/
└── .dsh/                          ← 文件夹名前面有点
    └── skills/                    ← 专门存放所有 skill
        └── crud-feature/          ← 每个 skill 一个以名称命名的文件夹
            └── SKILL.md           ← 具体文档叫 SKILL.md
```

逐条确认：

- **先有 `.dsh` 文件夹，前面有点。** 这是 DSH 的配置/扩展目录，点前缀表示隐藏目录，和 `.git`、`.claude` 同理。
- **下面有 `skills/` 文件夹，专门放所有 skill。**
- **每个 skill 一个以 skill 名称命名的独立文件夹。** 比如 `crud-feature/`、`release-checklist/`。
- **文件夹内的具体文档通常叫 `SKILL.md`，但这不是唯一允许的写法。**

### 一个完整示例

```text
todo-app/
├── .dsh/
│   └── skills/
│       ├── crud-feature/
│       │   └── SKILL.md
│       ├── code-review/
│       │   ├── SKILL.md
│       │   └── templates/
│       │       └── review-report.md
│       └── release-checklist/
│           └── SKILL.md
├── docs/
├── 01_Project/
├── AGENTS.md
└── ...
```

对应调用：

- 用户说“新增一个按优先级排序的功能” → 匹配 `crud-feature` 的 `description`，加载它的 `SKILL.md`。
- 用户说“帮我审查这段代码” → 匹配 `code-review`，加载其 `SKILL.md`，需要时引用 `templates/review-report.md`。
- 用户说“准备发版” → 匹配 `release-checklist`。

---

### 两个常见坑

1. **不要把 `SKILL.md` 直接放在 `skills/` 根下**，比如 `.dsh/skills/SKILL.md`。发现器按 `<name>/SKILL.md` 或 `<name>.md` 找，根下裸文件不会被当成某个具名 skill。
2. **不要小写 `skill.md`**。约定是大写 `SKILL.md`；小写可能不被识别，至少不符合约定，容易出问题。

### 补充：`SKILL.md` 不是唯一文件名

DSH 的技能发现器支持两种形式：

**形式一：目录形式（推荐）**

```text
.dsh/skills/crud-feature/SKILL.md
```

规则是 `<root>/<name>/SKILL.md`，**一层深度**。`SKILL.md` 必须放在以技能名命名的文件夹内，不能直接散落在 `skills/` 根目录下。

**形式二：扁平文件形式（不推荐）**

```text
.dsh/skills/crud-feature.md
```

规则是 `<root>/<name>.md`，直接放一个以技能名命名的 `.md` 文件在 `skills/` 下。

两种形式 DSH 都能识别。但**推荐用目录形式**，原因：

1. 目录形式可以附带 `scripts/`、`templates/`、`references/` 等辅助文件，扁平形式不行。
2. 目录形式是主流约定，`SKILL.md` 大写也是约定，工具和文档默认按这个找。
3. 目录形式更容易和 Git、版本管理配合，一个 skill 一个目录，边界清楚。

## 怎么用这份文件？

### 自动触发（主要方式）

Agent 在会话启动时，**只扫描所有 skill 的元数据（`name` 和 `description`）**，这个开销很小（约 500 tokens）。当你的输入匹配到某个 `description` 时，Agent 才会**按需加载该 `SKILL.md` 的完整正文**。这就是“渐进式披露”机制：不用的 skill 几乎不消耗上下文，用到的才完整读入。相比把所有规则全量塞进 `AGENTS.md`，初始 token 消耗可降低 95% 以上。

所以，你不需要在 `AGENTS.md` 里写“请遵循某某流程”，只需要确保 `description` 写得足够精准，Agent 会在相关时自动加载。你可以在 `AGENTS.md` 里用一句话指明“涉及发版时，加载 `release-checklist` skill”，作为补充提醒。

### 手动调用

你也可以在对话中直接用 `/skill-name` 来强制调用某个 skill。比如 `/crud-feature 新增一个按优先级排序的功能`。

---

## 与现有文档体系的关系

| 文档                                                                      | 角色                           | 与 `SKILL.md` 的关系                                                                                                 |
| ------------------------------------------------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| `AGENTS.md`                                                             | 每次会话必读的**短规则** | 当其中某段规则演变成多步骤程序时，**迁出**到 `SKILL.md`，`AGENTS.md` 只留触发提示                            |
| `plan.md` / `architecture.md` / `contracts.md` / `conventions.md` | **陈述性事实**（是什么） | `SKILL.md` 的步骤中**引用**它们，不复制内容。例如步骤写“读 `contracts.md` 定义类型”，而不是把字段表抄一遍  |
| `checklist.md`                                                          | **验收判据**             | `SKILL.md` 的“自查”步骤引用 `checklist.md`，不重复判据                                                           |
| `tasks.md` / `PROGRESS.md`                                            | **当前执行状态**         | `SKILL.md` 是**可复用的流程模板**，不记录某一次具体任务的状态；每次执行 skill 产生的进度仍写入 `PROGRESS.md` |

**一句话区分**：**`AGENTS.md` 管“每次必须知道什么”，`SKILL.md` 管“做某类事时按什么步骤做”，现有 docs 文档管“事实和状态是什么”。** 三者各司其职，不互相复制。

## 项目简单时要不要写 `SKILL.md`？

如果项目很小，你几乎没有重复性的多步骤流程，或者流程短到一句话就能说完，那**不必刻意创建 `SKILL.md`**。先让规则留在 `AGENTS.md`，流程留在你的对话习惯里。当“重复粘贴同一套步骤”成为真实痛点时，再抽出来写成 skill。`SKILL.md` 的价值来自复用，没有复用需求的 skill 只会增加维护负担。
