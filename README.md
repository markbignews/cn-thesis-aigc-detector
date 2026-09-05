# cn-thesis-aigc-detector · paper-mode：DeepSeek Harness 论文模式（开源技能包）

> **为 DeepSeek Harness（DSH）用户开源**：本仓库即 DSH 技能 `paper-mode` 的完整源码
> ——克隆到技能目录即可让 DSH Agent 获得"中文学术论文 AIGC 痕迹检测与降率改写"能力。
> 仓库布局与 DSH 技能目录完全一致（`SKILL.md` + `references/` + `scripts/` + `samples/`），
> 全部脚本零第三方依赖，脱离 DSH 也可独立使用。

功能一句话：Agent 按知网 CNKI AIGC 检测的**语言特征**估算论文 AI 率、给出逐段修改意见、
迭代改写把 AI 率降到目标比例（如 ≤10%），并支持 `.docx/.pptx/.xlsx/.pdf` 文档与定稿 Word 精确导出。

> ⚠️ **重要声明（请先阅读）**
>
> 1. 本技能输出的是基于知网 CNKI AIGC 检测**语言特征**的**估算**，不是知网/维普/Turnitin 等官方检测结果。官方检测才是最终标准。
> 2. **仅用于**作者在投稿/查重前对 AI 辅助生成内容做自查与人工润色，恢复自然、差异化的学术语体。**不得**用于规避学校或期刊的官方检测、掩盖代写、伪造数据。使用者须遵守所在机构关于生成式 AI 使用的规定并如实披露。
> 3. 改写过程中**不得编造**数据、实验、案例、引用；没有证据支撑时，宁可弱化语气、补充边界。

## 一、给 DSH 用户：安装与使用

### 安装（一行命令）

```bash
git clone https://github.com/noneedloong/cn-thesis-aigc-detector ~/.dsh/skills/paper-mode
```

或保留本地副本用软链（便于 `git pull` 更新原仓库后即时生效）：

```bash
git clone https://github.com/noneedloong/cn-thesis-aigc-detector ~/paper-mode-src
ln -s ~/paper-mode-src ~/.dsh/skills/paper-mode
```

### 使用方法

安装后，在 DSH 对话中直接提出需求即可，Agent 会自动加载本技能（`skill name: paper-mode`）：

> "检查这篇论文的 AI 率" / "降 AI 率到 10% 以下" / "这段改得不像 AI 写的" / "论文模式"

支持两种输入：**粘贴文本**，或**把文档放进会话工作区后以 `@路径` 引用**（Word/PPT/Excel/PDF 均可，由内置脚本自动提取）。

技能工作闭环（详见 [`docs/thesis_workflow_zh.md`](docs/thesis_workflow_zh.md)）：

1. 引述/引用可靠性核查（独立核验，先于检测）；
2. AI 率检测估算——每次由**不带会话历史的全新独立 Agent** 判定，避免"改写着自测"的乐观偏差；
3. 逐段修改意见，**作者确认后才动手**；
4. 按信号库改写（只动句法少动词汇、打散句长节奏、拆模板段、保留真实证据与学术语体）；
5. 复查循环：信号重扫 + 独立 Agent 再估算，直到 ≤ 目标比例（默认 ≤10%）；
6. 定稿**逐字**精确导出 `.docx`（`scripts/docx_write.py`），并提醒上官方平台实测。

判定与改写细则见 [`references/aigc_signals_zh.md`](references/aigc_signals_zh.md)：
5 大检测维度（L1 句法节奏 / L2 信息密度 / L3 术语句法位置 / L4 连接词重叠 / L5 模板段）、
8 条预防性写作规则、11 条修复策略、硬约束自检表、噪声预算（每千字保留 2–3 处轻微 AI 特征）。

### 更新

```bash
git -C ~/.dsh/skills/paper-mode pull
```

### 卸载

```bash
rm ~/.dsh/skills/paper-mode   # 软链安装时；git clone 安装则 rm -rf 该目录
```

## 二、目录结构（= DSH 技能目录布局）

```
cn-thesis-aigc-detector/            ← clone 到 ~/.dsh/skills/paper-mode
├── SKILL.md                        # 技能说明 + DSH 元数据（name: paper-mode）
├── references/
│   └── aigc_signals_zh.md          # AIGC 信号库 v2：判定与改写的唯一依据
├── scripts/                        # 全部零第三方依赖
│   ├── ai_signal.py                # 确定性信号扫描（.docx/.txt/.md/stdin，150–450 字分块）
│   ├── office_extract.py           # docx/pptx/xlsx 统一文本提取
│   ├── docx_extract.py             # docx 结构化原文提取
│   ├── docx_write.py               # 定稿文本精确导出 .docx（逐字保留）
│   ├── pdf_to_text.py              # PDF 文字版提取（pypdf → pdftotext 回退）
│   └── pdf_to_images.swift         # PDF 逐页转 PNG（macOS，公式/图表/扫描件视觉版）
├── samples/
│   ├── sample_ai_style.txt         # 示例：典型 AI 风格段落（可直接演示扫描）
│   └── sample_ai_style.docx        # 同内容的 Word 版
└── docs/
    └── thesis_workflow_zh.md       # 检测—改写闭环流程方法论（人/Agent 均可执行）
```

## 三、脱离 DSH：命令行直接使用

仅需 `python3`（≥3.8，标准库；PDF 文字版可选装 `pypdf` 或 poppler）。

```bash
# 信号扫描（.docx / .txt / .md，或 - 从 stdin 读）
python3 scripts/ai_signal.py 论文.txt
python3 scripts/ai_signal.py 论文.docx
python3 scripts/ai_signal.py - < 论文.txt

# 提取 Office 文档正文
python3 scripts/office_extract.py 论文.docx out.txt
python3 scripts/docx_extract.py 论文.docx out.txt

# PDF：文字版优先；视觉版（macOS）生成逐页 PNG 供人工/视觉模型判读
python3 scripts/pdf_to_text.py paper.pdf out.txt
swift scripts/pdf_to_images.swift paper.pdf outdir 1600

# 定稿精确导出 .docx（支持 # 标题、**加粗**，不重写内容）
python3 scripts/docx_write.py 定稿.txt 定稿.docx

# 试用样例
python3 scripts/ai_signal.py samples/sample_ai_style.txt
```

> 脚本只输出"信号证据"，不做最终判定。完整方法论与执行步骤见
> [`docs/thesis_workflow_zh.md`](docs/thesis_workflow_zh.md)；判定细则以信号库为准。

## 四、开源许可与致谢

[MIT](LICENSE) © noneedloong，2025。

方法论与判定信号在实战中参考并整合了以下 MIT 开源项目的经验（详见 `references/aigc_signals_zh.md` 第十节）：

- [qingshanliuci/cnki-aigc-skill](https://github.com/qingshanliuci/cnki-aigc-skill)
- [redbaronyyyyy-eng/humanizer-zh-academic](https://github.com/redbaronyyyyy-eng/humanizer-zh-academic)
- [ChHsiching/chhsich-thesis-aigc-skills](https://github.com/ChHsiching/chhsich-thesis-aigc-skills)
