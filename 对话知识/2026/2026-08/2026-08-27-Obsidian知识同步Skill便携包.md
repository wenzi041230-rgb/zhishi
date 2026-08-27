---
type: 对话知识
created: 2026-08-27 20:18
updated: 2026-08-27 20:18
source: Codex 对话
status: 部分确认
tags:
  - 对话知识
  - Obsidian
  - GitHub
  - Codex-Skill
  - 便携安装
---

# Obsidian 知识同步 Skill 便携包

## 用户目标

把当前使用的 Obsidian/GitHub 对话知识同步 Skill 打包，供另一台 Windows 电脑安装使用。

## 已确认结论

- 已制作不包含 Obsidian 笔记、Git 凭据、本机用户路径和当前仓库地址的便携压缩包。
- 包含 Skill 主体、界面元数据、知识库配置模板、知识检索与 wikilink 校验脚本、安装脚本、全局规则片段和中文安装说明，共 9 个文件。
- Skill 结构校验通过；4 个 PowerShell 脚本均可解析；压缩包全部条目可完整读取。
- 安装器已在隔离的临时 Codex 目录完成演练，验证了路径/远程/分支写入、知识检索冒烟测试、同名 Skill 备份和全局规则安装。

## 决策与依据

- 不直接复制当前 Skill 中的本机绝对路径，而由安装器在目标电脑写入新的知识库路径、Git 远程和分支，避免跨电脑路径失效。
- 新电脑可用 `-CloneVault` 克隆知识库；已有本地仓库时不克隆，只校验 `origin` 和当前分支。远程或分支不一致时停止，避免误推送。
- 全局“任务前检索、完成后沉淀”规则通过 `-AddGlobalRule` 显式启用；修改现有 `AGENTS.md` 前先备份，并用标记防止重复添加。
- 压缩包只作为本地交付物，不加入知识库 Git 仓库；知识库仅记录方法、验证结果和边界。

## 操作与产物

- 便携包：`08_Codex/deliverables/obsidian-chat-memory-portable-20260827.zip`
- SHA-256：`FE91CA5BEC17B604B3CC932E40BE6E5C6B08CABB46E45FF4208CABA7A6A9DDA4`
- 安装入口：解压后运行 `install.ps1`，提供 `VaultPath`、`RemoteUrl` 和 `Branch`。
- 安装说明：压缩包根目录的 `安装说明.md`。

## 待确认或待实测

- 尚未在第二台真实电脑上验证 Codex 版本、Git/Git Credential Manager 登录状态及 GitHub 推送权限。
- 安装后的检索冒烟测试只证明本地脚本能读到 Markdown；仍需在新 Codex 任务中验证隐式调用、知识写回、精确暂存、提交和推送全流程。

## 可复用知识

- PowerShell 中原生命令输出经过管道后，`$LASTEXITCODE` 可能不再保持可用值；应先把外部命令输出保存到数组并立即捕获退出码，再做 `Select-Object` 等管道处理。
- 跨电脑 Skill 不应把用户目录和 vault 路径写死在脚本默认参数中；配置模板加安装期替换更容易审计和迁移。
- 可移植性验证至少覆盖：Skill 元数据校验、脚本语法、真实安装演练、配置占位符清零、核心脚本冒烟测试、压缩包逐条读取和敏感字符串扫描。

## 关联知识

- [[对话知识/2026/2026-08/2026-08-12-建立Obsidian对话知识记忆工作流]]
- [[对话知识/00_对话知识索引]]
