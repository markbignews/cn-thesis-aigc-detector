# cn-thesis-aigc-detector · paper-mode 跨平台版本与方法论仓库

paper-mode（论文模式）技能：检测中文学术论文的 AI/AIGC 生成痕迹（按知网 CNKI AIGC 检测的**语言特征**估算 AI 率）、引述可靠性核查先行、给出逐段修改意见，并迭代改写把 AI 率降到目标比例（默认 ≤10%）。

本仓库是该技能的**跨平台分发与方法论仓库**，不再直接作为 DSH 技能包安装（DSH 版已拆分至独立仓库）。请按需取用：

| 你需要 | 位置 |
|---|---|
| **DSH 版技能包**（DeepSeek Harness 安装） | 独立仓库 [markbignews/dsh-paper-mode](https://github.com/markbignews/dsh-paper-mode)（唯一真源） |
| **WorkBuddy 版**（含上架用 zip） | 本仓库 [`workbuddy/`](workbuddy/README.md) |
| **通用版**（Codex / Claude Code 等支持 `SKILL.md` 的宿主） | 本仓库 [`generic/paper-mode/`](generic/paper-mode/SKILL.md) |
| 共享方法论资产（信号库/流程文档/命令行脚本/样例） | 本仓库根 `references/` `docs/` `scripts/` `samples/` |

> ⚠️ **声明（请先阅读）**
>
> 1. **非官方项目**：社区独立实现，与杭州深度求索（DeepSeek）及 deepseek-ai 官方团队无隶属、背书或合作关系。
> 2. 输出为基于知网 CNKI AIGC 检测**语言特征**的**估算**，不是知网/维普/Turnitin 官方检测结果；官方检测才是最终标准。
> 3. **仅用于**作者在投稿/查重前对 AI 辅助内容做自查与人工润色。**不得**用于规避学校或期刊的官方检测、掩盖代写、伪造数据；改写不得编造数据、实验、案例与引用。

## 目录结构

```
cn-thesis-aigc-detector/
├── references/
│   ├── aigc_signals_zh.md        # 信号库 v2：五维信号（L1–L5）、8 条预防性规则、11 条修复策略、自检表、噪声预算
│   └── duplicate_check_zh.md     # 查重参考库：机制口径、报告指标解读、降重策略、与降 AI 率协同（估算非官方）
├── scripts/                      # 六个零依赖脚本（python3 ≥3.8 标准库即可运行；swift 需 macOS）
│   ├── ai_signal.py              # 信号扫描（.docx/.txt/.md/stdin）
│   ├── office_extract.py         # docx/pptx/xlsx 统一提取
│   ├── docx_extract.py           # docx 结构化原文提取
│   ├── docx_write.py             # 定稿精确导出 .docx
│   ├── pdf_to_text.py            # PDF 文字版提取（可选装 pypdf/pdftotext）
│   └── pdf_to_images.swift       # PDF 逐页转 PNG（视觉版）
├── samples/                      # 演示样例（sample_ai_style.txt / .docx）
├── docs/
│   └── thesis_workflow_zh.md     # 闭环流程方法论（从拿到论文到交付定稿）
├── workbuddy/                    # WorkBuddy 适配版技能包（paper-mode/ + 上架用 zip，见 workbuddy/README.md）
├── generic/
│   └── paper-mode/               # 通用版技能包（平台无关；多 agent 首选 + 单 agent 降级盲评）
└── README.md / LICENSE / .gitignore   # 仓库级文件
```

三个平台版本的 `SKILL.md` **同方法论同源**：流程、红线、信号库完全一致，正文只按各自平台机制编写（DSH 依赖 `subagent` 子代理机制；WorkBuddy 走能力协商；通用版双模式）。各包内的 `references/`、`docs/`、`scripts/` 为同一份拷贝——**改方法论先改本仓库根目录资产，再同步各版本副本与 dsh-paper-mode 仓库**。

## 脱离 agent：命令行直接使用（共享资产）

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

> 脚本只输出"信号证据"，不做最终判定；判定细则以 `references/aigc_signals_zh.md` 为准，完整流程见 `docs/thesis_workflow_zh.md`。

## 开源许可与致谢

[MIT](LICENSE) © markbignews，2026。

方法论与判定信号在实战中参考并整合了以下 MIT 开源项目的经验（详见 `references/aigc_signals_zh.md` 第十节）：

- [qingshanliuci/cnki-aigc---skill](https://github.com/qingshanliuci/cnki-aigc---skill)
- [redbaronyyyyy-eng/humanizer-zh-academic](https://github.com/redbaronyyyyy-eng/humanizer-zh-academic)
- [ChHsiching/chhsich-thesis-aigc-skills](https://github.com/ChHsiching/chhsich-thesis-aigc-skills)
