# paper-mode for WorkBuddy（WorkBuddy 版）

本目录是论文模式技能（paper-mode）为 **WorkBuddy（腾讯办公智能体）** 适配的版本，存放于 `~/.workbuddy/skills/<技能名>/SKILL.md` 的约定目录结构下。

> 本仓库（cn-thesis-aigc-detector）是该技能的**唯一上游**：仓库根 `SKILL.md` 为 DeepSeek Harness（DSH）专用版，本目录为 WorkBuddy 适配版（含可直接上传的 `paper-mode-skillhub.zip`）。此前单独发布的独立仓库已并入本仓库，请勿再按旧地址安装。

> ⚠️ **版本状态（请先阅读）**：本版按 WorkBuddy 技能约定与社区教程（[WorkBuddy 技能系统深度指南](https://cloud.tencent.com.cn/developer/article/2693324)、[WorkBuddy Skill 制作实操指南](https://cloud.tencent.com.cn/developer/article/2725219)）做格式与流程适配，**未经真实 WorkBuddy 运行环境实测**；原版仅针对 DeepSeek Harness 开发与测试。使用中发现偏差，按 `paper-mode/SKILL.md` 末尾「能力协商」自行调整即可。
> 与本仓库其他部分一样：输出为基于知网 AIGC 语言特征的**估算**，非官方检测结果；仅限合规自查与润色用途，不得用于规避官方检测、掩盖代写或学术不端。

## 目录

```
workbuddy/
├── paper-mode/                    ← 技能包：复制为 ~/.workbuddy/skills/paper-mode/
│   ├── SKILL.md                   # 技能正文（frontmatter: name/description/description_zh/description_en/display_name，对齐技能市场格式）
│   ├── references/aigc_signals_zh.md   # 信号库（与仓库根同源同一份）
│   ├── docs/thesis_workflow_zh.md      # 流程方法论（与仓库根同源同一份）
│   └── scripts/                   # 可选加速脚本（环境可运行 python3 时使用）
└── paper-mode-skillhub.zip        # 上架/App 内导入用（内容与 paper-mode/ 一致，随仓库更新）
```

## 安装（任选其一）

```bash
# 方式一：拷贝（最直接）
cd cn-thesis-aigc-detector/workbuddy
mkdir -p ~/.workbuddy/skills
cp -R paper-mode ~/.workbuddy/skills/        # 技能目录名保持 paper-mode
ls ~/.workbuddy/skills/paper-mode/SKILL.md   # 验证安装
```

方式二：在 WorkBuddy App 内「添加技能 → 上传技能」，导入本目录的 `paper-mode-skillhub.zip`（与 `paper-mode/` 内容一致，可直接下载）。

提示：`~/.workbuddy` 不存在时先启动一次 WorkBuddy 让其生成；无需重启，对话中描述需求即可按 description 自动匹配，也可直接说"用论文模式技能"。

## 更新

```bash
rm -rf ~/.workbuddy/skills/paper-mode
cp -R cn-thesis-aigc-detector/workbuddy/paper-mode ~/.workbuddy/skills/
```

仓库每次更新时，`paper-mode-skillhub.zip` 会同步重建，App 内重新上传即可。

## 与 DSH 版的差异（方法论同源）

- 正文不再引用 DeepSeek Harness 专有机制（子代理、上传插件、文档生成插件等），改为「能力协商」：宿主具备什么能力就用什么，不具备则如实说明并降级执行（纯文本输入、人工逐块判定、文本交付）。
- 流程与红线与 DSH 版 **v2 对齐**：引述核查先行（角色隔离、分批核验）→ 独立 AI 率检测 → 逐段修改意见（确认后改）→ 复查循环 → 交付并建议官方实测。
- `references/`、`docs/`、`scripts/` 与仓库根版本共享同一份平台无关内容（拷贝而非另写），保证判定标准一致。
- frontmatter 采用技能市场格式字段（`name`/`description`/`description_zh`/`description_en`/`display_name`），便于商店展示与自动匹配。

## 许可

[MIT](../LICENSE) © markbignews。第三方技能市场（如 SkillHub/ClawHub）的上架条款可能另行规定（如 ClawHub 统一 MIT-0），以各平台约定为准。
