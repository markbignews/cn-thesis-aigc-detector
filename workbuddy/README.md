# WorkBuddy 版（paper-mode for WorkBuddy）

本目录是论文模式技能为 **WorkBuddy（腾讯办公智能体）** 适配的版本，存放于 `~/.workbuddy/skills/<技能名>/SKILL.md` 的约定目录结构下。

> ⚠️ **版本状态（请先阅读）**：本版按 WorkBuddy 技能约定与社区教程（[WorkBuddy 技能系统深度指南](https://cloud.tencent.com.cn/developer/article/2693324)、[WorkBuddy Skill 制作实操指南](https://cloud.tencent.com.cn/developer/article/2725219)）做格式与流程适配，**未经真实 WorkBuddy 运行环境实测**；原版仅针对 DeepSeek Harness 开发与测试。使用中发现偏差，按 `paper-mode/SKILL.md` 末尾「能力协商」自行调整即可。
> 与本仓库其他部分一样：输出为基于知网 AIGC 语言特征的**估算**，非官方检测结果；仅限合规自查与润色用途。

## 目录

```
workbuddy/
└── paper-mode/                    ← 技能包：复制为 ~/.workbuddy/skills/paper-mode/
    ├── SKILL.md                   # 技能正文（frontmatter: name/description/agent_created）
    ├── references/aigc_signals_zh.md   # 信号库（与仓库根同源）
    ├── docs/thesis_workflow_zh.md      # 流程方法论（与仓库根同源）
    └── scripts/                   # 可选加速脚本（环境可运行 python3 时使用）
```

## 安装

```bash
cd cn-thesis-aigc-detector/workbuddy
mkdir -p ~/.workbuddy/skills
cp -R paper-mode ~/.workbuddy/skills/        # 技能目录名保持 paper-mode
ls ~/.workbuddy/skills/paper-mode/SKILL.md   # 验证安装
```

提示：`~/.workbuddy` 不存在时先启动一次 WorkBuddy 让其生成；无需重启，对话中描述需求即可按 description 自动匹配，也可直接说"用论文模式技能"。

## 更新

```bash
rm -rf ~/.workbuddy/skills/paper-mode
cp -R cn-thesis-aigc-detector/workbuddy/paper-mode ~/.workbuddy/skills/
```

## 与 DSH 版的差异

- 正文不再引用 DeepSeek Harness 专有机制（子代理、上传插件、文档生成插件等），改为「能力协商」：宿主具备什么能力就用什么，不具备则如实说明并降级执行（纯文本输入、人工逐块判定、文本交付）。
- `references/`、`docs/`、`scripts/` 与仓库根版本共享同一份平台无关内容（拷贝而非另写），保证判定标准一致。
- `agent_created: true` 已写入 frontmatter，WorkBuddy 可在对话中代为修改本技能。
