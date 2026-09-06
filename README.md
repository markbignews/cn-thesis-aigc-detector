# cn-thesis-aigc-detector · paper-mode：中文学术论文 AIGC 痕迹检测与降率改写

面向**DeepSeek Harness（DSH）用户**开源的社区技能：检测中文学术论文的 AI/AIGC 生成痕迹（按知网 CNKI AIGC 检测的语言特征估算 AI 率）、给出逐段修改意见，并迭代改写把 AI 率降到目标比例（如 ≤10%）。支持 Word(.docx)/PowerPoint(.pptx)/Excel(.xlsx)/PDF 文档与定稿 Word 精确导出。

本仓库**同时**是一个可直接克隆安装的 DSH skill bundle（`paper-mode`），也是一个可脱离 DSH 独立使用的零依赖命令行工具箱。

> ⚠️ **声明（请先阅读）**
>
> 1. **非官方项目**：本仓库是社区独立实现，与杭州深度求索（DeepSeek）及 deepseek-ai 官方团队无隶属、背书或合作关系，不声称是 DeepSeek 官方出品或官方推荐。
> 2. 输出为基于知网 CNKI AIGC 检测**语言特征**的**估算**，不是知网/维普/Turnitin 官方检测结果；官方检测才是最终标准。
> 3. **仅用于**作者在投稿/查重前对 AI 辅助内容做自查与人工润色。**不得**用于规避学校或期刊的官方检测、掩盖代写、伪造数据。使用者须遵守所在机构关于生成式 AI 的规定并如实披露；改写不得编造数据、实验、案例与引用。
> 4. 安装/使用细节以 DeepSeek Harness 官方文档为准（见文末[官方依据](#官方依据)），本 README 中的路径与格式均按官方文档核对。

## 适配范围与测试状态

- **专门适配 DeepSeek Harness（DSH）**：本仓库的 `SKILL.md` 是 DSH 专用技能正文，依赖 DSH 的运行机制（会话技能加载、子代理、工作区文件读写、web 检索、文档生成类插件等），并按 DSH 官方 skill 规范编写。
- **仅在 DSH 上开发与测试**：本项目从开发到验证均在 DeepSeek Harness 环境完成，**未在其他任何 agent 工具（Claude Code、Cursor、Cherry Studio、Codex 及其他通用 agent/框架）上测试或验证过**，不保证在 DSH 之外的可用性。
- 若要在其他平台使用，请知悉：`scripts/`、`references/`、`docs/` 为平台无关资产可自行取用；`SKILL.md` 的机制与工具名需要自行适配，适配后未经 DSH 之外实测的兼容性由使用者自行承担。
- 发现问题或做了平台适配，欢迎在 Issues 反馈（但不承诺支持 DSH 之外的运行环境）。

## 功能

| 模块 | 能力 | 入口 |
|---|---|---|
| 文本提取 | `.docx/.pptx/.xlsx` 正文提取（纯 stdlib 零依赖）；`.docx` 结构化查看 | `scripts/office_extract.py` / `scripts/docx_extract.py` |
| PDF 解析 | 文字版（pypdf → pdftotext 回退）；公式/图表/扫描件视觉版转图 | `scripts/pdf_to_text.py` / `scripts/pdf_to_images.swift` |
| 信号扫描 | 确定性启发式扫描：句长节奏、连接词堆叠、模板套话、缓和语、引用/数据信号，150–450 字分块输出证据 | `scripts/ai_signal.py` |
| 判定依据 | 5 大检测维度（L1–L5）、8 条预防性写作规则、11 条修复策略、硬约束自检表、噪声预算 | `references/aigc_signals_zh.md` |
| 流程方法 | "引述核查 → 检测估算 → 修改意见 → 确认改写 → 复查循环 → 精确导出"闭环 | `docs/thesis_workflow_zh.md` |
| 精确导出 | 定稿文本**逐字**装回 `.docx`（数字/术语/公式/引号保留，支持 `#` 标题与 `**加粗**`） | `scripts/docx_write.py` |

## 一、给 DSH 用户：安装与使用

### 1.1 本仓库符合的官方 skill 规范（摘要）

以下内容出自官方文档 [dsh-skill-filesystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/skill/skill-filesystem/README.zh.md) 与 [skill 子系统参考](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/skills.zh.md)：

- **身份**：skill 名称为 kebab-case（`^[a-z0-9]+(?:-[a-z0-9]+)*$`）；本技能 = `paper-mode` ✅。
- **格式**：被扫描根目录**顶层**的目录 bundle `<name>/SKILL.md`（或平铺 `<name>.md`）；本仓库即一个目录 bundle，frontmatter `name`/`description` 为必填，另有可选 `whenToUse`、`metadata`、`disable-model-invocation`、`user-invocable`。
- **目录身份一致**：候选 skill 名取自**目录名**（`<name>/SKILL.md`），加载时若正文 frontmatter `name` 与候选名不再匹配会被拒绝——因此**克隆目录必须命名为 `paper-mode`**（见 1.3）。
- **描述上限**：模型会话目录只渲染 `name` + `description`（默认上限 500 字符，官方配置 `catalogDescriptionMaxLength`）；本技能 description 243 字符 ✅。

### 1.2 发现根目录与优先级（官方表）

| Rank | 来源 | 路径 |
|---|---|---|
| 100 | 项目级 | `<projectRoot>/.dsh/skills` |
| 200 | 项目级 | `<projectRoot>/.agents/skills` |
| 300 | 自定义 | `Config.customSkillDirs` |
| 400 | 用户级 | `<dshHome>/skills`（`dshHome` = `$DSH_HOME` 或 `~/.dsh`） |
| 500 | 用户级 | `<agentsHome>/skills`（`agentsHome` = `$DSH_AGENTS_HOME` 或 `~/.agents`） |

项目根目录 = 含 `.git` 的最近祖先目录。rank 越小越优先；本仓库默认推荐安装到 **rank 400（用户级）**，随项目分发则装到 rank 100（如团队把 `.dsh/skills/paper-mode` 纳入项目 git 仓库）。

### 1.3 安装

```bash
# 推荐：用户级安装（dshHome 默认 ~/.dsh）
git clone https://github.com/markbignews/cn-thesis-aigc-detector ~/.dsh/skills/paper-mode

# 或项目级安装（随项目 git 分发，团队成员 clone 项目即得技能）
mkdir -p <projectRoot>/.dsh/skills
git clone https://github.com/markbignews/cn-thesis-aigc-detector <projectRoot>/.dsh/skills/paper-mode
```

> ⚠️ 克隆目标的**目录名必须是 `paper-mode`**：官方按目录名解析候选 skill（`<name>/SKILL.md`），frontmatter `name` 与候选名不一致的定义会被拒绝。也**不要**把仓库内容直接铺进 `~/.dsh/skills/`（那会把本仓库目录 `cn-thesis-aigc-detector` 自身当成 bundle 名，与 frontmatter `name: paper-mode` 冲突）。若设置了 `$DSH_HOME`，请替换 `~/.dsh` 为对应目录。

验证：

```bash
ls ~/.dsh/skills/paper-mode/SKILL.md   # 存在即安装就位
```

本地提供方**监视**各扫描根目录，新增 skill 无需重启，会在**下一个模型步骤的会话目录**中自动出现（官方热刷新机制）。

### 1.4 使用

- **模型侧**：会话目录仅向模型展示 `name` 与 `description`，模型在任务匹配时通过 `skill` 工具按名加载正文。直接说需求即可触发，例如：
  > "检查这篇论文的 AI 率" / "把 AI 率降到 10% 以下" / "这段改得不像 AI 写的"
- **用户侧**：官方支持用 `/name` 直接调用（本技能 `user-invocable` 默认开启）：`/paper-mode`。
- **输入**：粘贴文本，或把文档放进会话工作区后以 `@路径` 引用（Word/PPT/Excel/PDF 由技能脚本自动提取）。
- **工作闭环**：引述核查 → 独立检测估算 → 逐段修改意见（确认后才改）→ 复查循环 → 精确导出，详见 [`docs/thesis_workflow_zh.md`](docs/thesis_workflow_zh.md)。

### 1.5 更新 / 卸载 / 热刷新说明

```bash
# 更新
git -C ~/.dsh/skills/paper-mode pull
# 卸载
rm -rf ~/.dsh/skills/paper-mode     # （项目级安装则删除对应目录）
```

官方监视行为：`SKILL.md` 正文与 frontmatter 的修改在**下一模型步骤**刷新目录或影响后续加载，无需重启；`references/`、`scripts/` 等 bundle 资源子树的变更**不触发**目录刷新（目录 digest 不变），但正文每次加载都会重读文件，资源按相对引用随用随取。

### 1.6 调用策略（可选，改 frontmatter）

官方支持两个 kebab-case frontmatter 键控制调用面，省略即默认两侧都允许：

- `disable-model-invocation: true` — 从模型会话目录与加载器排除（仅用户可调用）；
- `user-invocable: false` — 从用户命令目录排除。

## 二、目录结构（= DSH skill bundle 布局）

```
cn-thesis-aigc-detector/          ← 安装为 <root>/paper-mode/
├── SKILL.md                      # 技能正文（DSH 版）：frontmatter（name/description/whenToUse/metadata）+ 指令
├── references/
│   └── aigc_signals_zh.md        # 信号库 v2：判定与改写的依据（按需相对引用加载）
├── scripts/                      # 六个脚本（python3 ≥3.8 标准库即可运行；swift 需 macOS）
│   ├── ai_signal.py              # 信号扫描（.docx/.txt/.md/stdin）
│   ├── office_extract.py         # docx/pptx/xlsx 统一提取
│   ├── docx_extract.py           # docx 结构化原文提取
│   ├── docx_write.py             # 定稿精确导出 .docx
│   ├── pdf_to_text.py            # PDF 文字版提取（可选装 pypdf/pdftotext；缺省输出引导）
│   └── pdf_to_images.swift       # PDF 逐页转 PNG（视觉版）
├── samples/                      # 演示样例（sample_ai_style.txt / .docx）
├── docs/
│   └── thesis_workflow_zh.md     # 闭环流程方法论
├── workbuddy/                    # WorkBuddy 适配版技能包（paper-mode/ + 上架用 zip，见 workbuddy/README.md）
└── README.md / LICENSE / .gitignore   # 仓库级文件（不影响技能发现）
```

说明：官方发现只认扫描根**顶层**的 `<name>/SKILL.md` 与 `<name>.md`；bundle 内的 `docs/`、`README.md`、`LICENSE` 等仓库级文件不会被当作 skill，也不影响目录。

仓库根 `SKILL.md`（DSH 版）与 `workbuddy/paper-mode/SKILL.md`（WorkBuddy 版）为**同源同步**的两个平台版本：方法论、红线、信号库（`references/`、`docs/`、`scripts/` 为同一份内容）完全一致，正文只按各自平台机制编写。

## 三、脱离 DSH：命令行直接使用

```bash
# 信号扫描（.docx / .txt / .md，或 - 从 stdin 读）
python3 scripts/ai_signal.py 论文.txt
python3 scripts/ai_signal.py 论文.docx

# 提取 Office 文档正文
python3 scripts/office_extract.py 论文.docx out.txt
python3 scripts/docx_extract.py 论文.docx out.txt

# PDF：文字版优先；视觉版（macOS）生成逐页 PNG 供人工/视觉模型判读
python3 scripts/pdf_to_text.py paper.pdf out.txt
swift scripts/pdf_to_images.swift paper.pdf outdir 1600

# 定稿精确导出 .docx
python3 scripts/docx_write.py 定稿.txt 定稿.docx

# 试用样例
python3 scripts/ai_signal.py samples/sample_ai_style.txt
```

> 脚本只输出"信号证据"，不做最终判定；完整方法论见 [`docs/thesis_workflow_zh.md`](docs/thesis_workflow_zh.md)，判定细则以信号库为准。

## 官方依据

本 README 的安装路径、格式与优先级均按 DeepSeek Harness 官方仓库（`deepseek-ai/deepseek-harness`，master）核对：

- [packages/skill/skill-filesystem/README.zh.md — 本地 skill 格式、frontmatter、根目录与监视](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/skill/skill-filesystem/README.zh.md)
- [docs/subsystems/skills.zh.md — skill 命名、发现优先级与目录/工具约定](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/skills.zh.md)
- [packages/skill/README.zh.md — skill 能力家族概览（用户 `/name` 调用）](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/skill/README.zh.md)

## 开源许可与致谢

[MIT](LICENSE) © markbignews，2026。

方法论与判定信号在实战中参考并整合了以下 MIT 开源项目的经验（详见 `references/aigc_signals_zh.md` 第十节）：

- [qingshanliuci/cnki-aigc---skill](https://github.com/qingshanliuci/cnki-aigc---skill)
- [redbaronyyyyy-eng/humanizer-zh-academic](https://github.com/redbaronyyyyy-eng/humanizer-zh-academic)
- [ChHsiching/chhsich-thesis-aigc-skills](https://github.com/ChHsiching/chhsich-thesis-aigc-skills)
