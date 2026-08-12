---
type: 对话知识
created: 2026-08-12 18:35
updated: 2026-08-12 18:52
source: Codex 对话
status: 部分确认
tags:
  - 对话知识
  - skill评估
  - Fusion360
  - CAD
---

# Fusion 360 Expert skill 本地冲突评估

## 用户目标

评估 GitHub 上 `theneoai/awesome-skills` 的 `fusion360-expert` 是否与本机已安装的 skills、EasyEDA bridge 和本地安装布局冲突。

## 已确认结论

- GitHub 版本为 `fusion360-expert` 1.0.0、MIT；内容是 `SKILL.md` 加 `references/` 下的 Markdown，未发现脚本、包依赖、环境变量、端口、MCP server 或 Fusion 360 控制接口。
- 本机没有同名目录或同名 skill；`C:\Users\CLX\.codex\skills` 的本地安装模式是指向 `E:\CodexSkills` 的 junction，因此可按 `E:\CodexSkills\fusion360-expert` + `C:\Users\CLX\.codex\skills\fusion360-expert` junction 的方式安装，不会覆盖现有目录。
- 现有相关 skill 包括 `easyeda-api`、`blender-digital-art`、`curves-surfaces-cagd` 和 `polygon-mesh-processing`。它们分别覆盖 EasyEDA 实时 API、Blender 资产、CAGD 曲线曲面算法和网格处理；与 Fusion 360 skill 是邻接/互补关系，不是文件或运行时冲突。
- EasyEDA bridge 仍由 `127.0.0.1:7655` 的 MCP 配置和 `49620-49629` 的本地 bridge 端口承担；Fusion 360 skill 不触碰这些配置。
- 已使用本机 skill 安装脚本将 GitHub skill 安装到 `E:\CodexSkills\fusion360-expert`，并建立 `C:\Users\CLX\.codex\skills\fusion360-expert` junction；安装内容为 `SKILL.md` 和 6 个 Markdown reference 文件。
- 本机 Fusion 360 本体已安装且当前正在运行；Fusion API 文件存在，版本为 `2703.1.11`。
- 当前 Fusion 360 用户 API 目录中的 `AddIns` 和 `Scripts` 均为空；Codex `config.toml` 没有 `fusion`/`autodesk`/`cad` MCP 条目，`C:\Users\CLX\.codex\mcp-servers` 也没有对应 server，当前工具列表没有 Fusion 360 MCP 工具。

## 决策与依据

建议：可以安装，但仅作为 CAD/CAM 对话辅助知识层，不要把它当成 Fusion 360 自动化工具，也不要覆盖现有 EDA/Blender/CAGD skill。

触发边界建议：

- 机械建模、装配、钣金、3D 打印、Fusion CAM：优先使用 `fusion360-expert`。
- EasyEDA 原理图、PCB、封装、实时桌面操作：继续使用本机 `easyeda-api`。
- NURBS、STEP/IGES 算法或曲面数学实现：继续使用本机 `curves-surfaces-cagd`，必要时与 Fusion 360 skill 组合。
- Blender 拓扑、UV、渲染和游戏资产：继续使用本机 `blender-digital-art`。

## 待确认或待实测

- 该 skill 没有实际 Fusion 360 API/桌面连接能力；是否要补充 Autodesk Fusion 360 API、脚本和本机 Fusion 安装状态，需另行验证。
- 原文中的工程参数不能直接作为制造规范：H7/g6 不是固定的 `±0.015 mm`，打印温度/支撑角度依设备、材料和工艺而变；CAM 进给、转速、刀具和余量必须依据材料、机床、刀具及制造商数据复核。
- 原文的 Shell 特征顺序存在前后表述不一致，`iProperties` 也更像 Inventor 术语；正式设计时应以 Autodesk Fusion 360 当前官方文档和实际模型验证为准。

## 可复用知识

- 评估第三方 skill 时先分离三类冲突：名称/安装路径冲突、运行时依赖冲突、知识/触发边界冲突。
- 对仅包含 Markdown 的 prompt skill，主要风险不是本地运行时冲突，而是错误或过时的领域建议被误当成工程规范。
- 本机 skill 保持“E: 实体目录、C: junction 暴露”的布局；安装前必须确认目标目录不存在，并保留现有 skill 与未提交改动。
- “Fusion 360 已安装”不等于“Fusion 360 MCP 已连接”；需要额外的 Fusion 360 Add-In/API bridge 和 Codex MCP server 配置，才能进行实时桌面模型读写。

## 操作与产物

- 安装产物：`E:\CodexSkills\fusion360-expert`；暴露路径：`C:\Users\CLX\.codex\skills\fusion360-expert`。
- 诊断边界：已核对 Codex MCP 配置、MCP server 目录、Fusion 进程、Fusion API 目录、用户 AddIns/Scripts 目录及相关运行时进程；未执行 Fusion 模型修改或安装第三方 MCP。

## 关联知识

- [[对话知识/2026/2026-08/2026-08-12-建立Obsidian对话知识记忆工作流|Obsidian 对话知识记忆工作流]]
- [GitHub fusion360-expert](https://github.com/theneoai/awesome-skills/tree/main/skills/tool/cad/fusion360-expert)
