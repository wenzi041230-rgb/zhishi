---
type: 对话知识
created: 2026-09-15 16:20
updated: 2026-09-15 18:56
source: Codex 对话
status: 已确认
tags:
  - 对话知识
  - Codex-Skill
  - 冲突评估
---

# mattpocock/skills 与本机 Skill 冲突评估

## 用户目标

检查 `https://github.com/mattpocock/skills` 当前版本是否会与本机已安装的 Codex skill、全局规则或安装布局冲突。

## 已确认结论

- 评估基准为仓库 `main` 当前抓取版本，commit `3cca18b368ae95cdbdebbff572ccafa662551015`，包含 37 个 `SKILL.md`。
- 本机活动 skill 入口为 `C:\Users\CLX\.codex\skills`，其中 973 个目录联接到实体源 `E:\CodexSkills`。按活动入口的顶层目录名比较，仓库只有一个同名项：`handoff`。
- `handoff` 是用户显式调用型 skill，本机已有版本与仓库版本语义相同但不是同一文件：两处文字有差异，SHA-256 分别为本机 `6436459FEDE941ECBE5C9DC742D66AB213F96C18BF25E935CD375BF61F247C11`、仓库 `96F8045F01F569DAA3737150D918B7201CCD090905FB10CEFEDD326E4001F551`。不建议直接覆盖本机版本。
- 没有发现同名的文件、端口、Python/Node 依赖、MCP 服务或硬件接口冲突。仓库 skill 目录中的非 Markdown 文件只有 Bash 模板/脚本和一个依赖巡检配置，主要由对应流程按需使用。

## 决策与依据

- 用户确认后已按仓库指定提交安装全部 37 个 skill。仓库说明 Codex 使用 `npx skills@latest add mattpocock/skills` 时可选择 skill；其 Claude 插件路线与可编辑文件路线是两种互斥安装方式。
- 本机官方安装器在目标目录已存在时会中止；此前的 `handoff` 冲突已通过备份旧实体目录、移除旧联接、安装仓库版并重建联接解决。
- 仓库维护脚本面向 `~/.agents/skills` 并创建符号链接；本机使用的是 `~/.codex/skills` 加 `E:\CodexSkills` 实体源，不能直接运行该维护脚本。脚本还会删除同名真实目录后再建链接，不能拿来覆盖本机 skill 源。

## 操作与产物

- 旧本机 `handoff` 已从活动目录移出，完整备份在 `E:\CodexSkills\.backups\handoff-local-20260915`；备份前后 `SKILL.md` SHA-256 一致。
- 仓库 commit `3cca18b368ae95cdbdebbff572ccafa662551015` 的 37 个 skill 已安装到 `E:\CodexSkills`，并建立 37 个 `C:\Users\CLX\.codex\skills` 目录联接。
- 逐文件比对 100 个安装文件，规范化换行符后与仓库检出内容一致；没有缺失文件、额外文件或错误联接。
- 已在 `C:\Users\CLX\.codex\AGENTS.md` 增加优先级约束：仓库 skill 在重叠能力中优先，本机 skill 仅作补充；系统/开发者规则、显式用户要求和 `obsidian-chat-memory` 仍然优先于该约束。
- 本机旧版 `quick_validate.py` 对 22 个带 `disable-model-invocation` 或 `argument-hint` 的双平台 skill 报前置字段警告；现有本机 `handoff` 也会报同类警告，未因此修改仓库元数据。

## 重点边界

- `domain-modeling` 与本机 `domain-driven-design` 有方法论重叠；前者偏 `CONTEXT.md`/ADR 和术语建模，后者偏 C++/Qt 的 DDD。两者可互补，但在同一任务中可能重复触发。
- `tdd` 与本机 `xunit-test-patterns`、`webapp-testing` 有测试流程重叠；可组合使用，但应指定一个作为主流程，另一个只补充测试设计或浏览器验证。
- `research` 与本机 `content-research-writer` 有研究和引用重叠；仓库版本还会把研究结果写入当前项目 Markdown，需避免与 Obsidian 知识库记录形成两套来源。
- `code-review`、`codebase-design` 与本机 `clean-code`、`code-complete`、`git-workflow-and-versioning` 是相邻能力，不是文件冲突；组合时需避免同一轮重复审查或重复 Git 收尾。
- `writing-for-agents` 与本机 `.system/skill-creator`、`plugin-creator`、`skill-share` 有技能/AGENTS 文档编写重叠；修改 skill 或规则文件时只保留一套规范作为最终裁决。
- `setup-matt-pocock-skills` 会在项目内写入 `docs/agents/*`、`CONTEXT.md`/ADR 相关配置并修改项目 `AGENTS.md` 或 `CLAUDE.md`。不要在 Obsidian 知识库根目录直接运行，以免引入另一套项目知识文档布局。
- `git-guardrails-claude-code` 会阻止 `git push`、`reset --hard`、`clean` 等命令。若启用为全局 Claude Code hook，它会阻断当前 Obsidian skill 必需的知识库提交/推送；除非明确需要，不安装或不设为全局。
- `wizard` 会生成可写入 `.env` 和 GitHub Secrets/Variables 的 Bash 向导；该能力应保持显式任务触发，并在生成或执行前检查目标路径和敏感变量边界。

## 可复用知识

- 第三方 skill 冲突检查应分成四层：顶层名称/安装路径、文件内容版本、隐式触发边界、运行时或项目写入行为。
- 对本机 skill 安装，优先使用 `E:\CodexSkills` 作为实体源、`C:\Users\CLX\.codex\skills` 作为 Codex 暴露入口；安装前先检查同名目录，不覆盖已有版本。
- 已安装整套仓库 skill；后续任务按全局 `AGENTS.md` 约束选择，重叠能力优先使用仓库版，必要时再调用本机版补充。

## 待确认或待实测

- 新安装的 skill 预计在 Codex 下一轮重新发现后可用；本次没有执行任何仓库 skill 的项目配置、Git hook、Issue 发布或密钥写入流程。
- 仓库后续更新可能改变 skill 名称、隐式触发标记或写入行为，重新安装前应按同一四层检查复核。

## 关联知识

- [[对话知识/2026/2026-08/2026-08-12-fusion360-expert本地冲突评估|Fusion 360 Expert skill 本地冲突评估]]
- [[对话知识/2026/2026-08/2026-08-27-Obsidian知识同步Skill便携包|Obsidian 知识同步 Skill 便携包]]
- [[对话知识/2026/2026-08/2026-08-12-建立Obsidian对话知识记忆工作流|建立 Obsidian 对话知识记忆工作流]]
