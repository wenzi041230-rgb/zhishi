---
type: 对话知识
created: 2026-11-18 20:52
updated: 2026-11-19 15:49
source: Codex 对话
status: 部分确认
tags:
  - 对话知识
  - EDA
---

# OrCAD 与 Allegro 安装包核验

## 已确认结论

- OrCAD Capture 负责原理图设计。PCB 放置与布线使用 Allegro 或 OrCAD PCB Editor。安装介质名称和附带的 Allegro 配置文件不足以确认包含 PCB Editor。
- 核对解压目录和原始压缩包清单均只找到 Capture 主程序及其辅助工具，缺少 PCB Editor。本次只做了静态检查，PCB Editor 未安装和实测。

## 可复用知识

- 核查旧版 Cadence 安装介质时，先查看压缩包清单及可执行文件的产品信息，确认包包含目标编辑器和适用授权。可通过 Cadence 官方渠道取得完整安装介质。
- 验收 PCB 编辑器时，启动后用测试板验证打开、放置、布线和另存，保留原工程。
- [Cadence Allegro X 官方试用申请](https://www5.cadence.com/allegro-x-free-trial.html)

## Capture 16.3 原理图验收

- 当 Windows 报应用程序的并行配置不正确时，查询 Application 日志的 SideBySide 事件。此处事件 33 指出缺少 x86 的 Microsoft.VC80.MFC。安装并验证微软签名的 [Visual C++ 2005 SP1 x86 MFC 更新 KB2538242](https://www.microsoft.com/en-us/download/details.aspx?id=26347) 后，Capture 16.3 p002 进入 Demo Edition。
- 已实测新建测试原理图、放置导线和将 DSN 写入磁盘。启动程序及测试文件留在目标数据盘。首次从开始菜单启动时停在产品选择框，随后完成复验。
- 新建时日志提示 CAP0036，精简包缺少 CAPSYM.OLB 和常用符号库。需要用兼容的 OLB 补齐元件来源。Demo Edition 有设计规模限制，复杂工程保存未测试。

## 2026-11-19 复验

- 旧版 Capture 进程正常响应时，如果窗口未显示，先检查是否有独立的 Cadence Product Choices 对话框。尝试将该对话框切到前台，选择 OrCAD Capture 并勾选 Use as default。退出后从开始菜单入口重新启动，实测直接进入主界面。
- 已实测打开测试 DSN、添加文字、保存退出，再通过关联的 OPJ 重新打开；原理图页保留了新增文字。默认符号库仍需单独补齐。

## GitHub 常用元件库

- 在公开候选中选择 [XiaoSenLuo/CadenceLibrary](https://github.com/XiaoSenLuo/CadenceLibrary/tree/dc3c2caac3ea6ee8727365a3b5f16b059a99ca48)。取其 Symbol 下的十个常用 OLB，涵盖电阻、电容、电感、二极管、LED、三极管、MOSFET、USB、2.54 mm 排针和振荡器。下载后校验各文件与 GitHub blob 一致。
- 在 Capture 16.3 的 Place → Part → Add Library 可一次多选多个 OLB。正常退出后，软件把路径写到 Capture.ini 的 Part Selector Configured Libraries。实测退出并重新启动后十个库仍能从 Place Part 看到。
- 用 0603 电阻和电容验证了放置、保存和重新打开。其他类别确认可在库面板中加载。该仓库的 OLB 用于原理图元件，未补齐 CAPSYM.OLB 或验收 PCB 封装。

## DSN 导出 EDIF

- 原理图导出用 OrCAD Capture 的 File → Export Design → EDIF，设置 .EDF 输出路径，配置文件选安装目录附带的 CAP2EDI.CFG。需要导出整份设计时，先在 Project Manager 中选中 .DSN 根节点。Tools → Create Netlist 下的 EDIF 是 .EDN 网表。
- 本次静态核对的精简包含 CAP2EDI.CFG，但缺少 Capture 可执行文件引用的 cap2edid.dll 与 cap2edll.dll。未实测生成 EDF。遇到导出组件加载错误时，使用带匹配 EDIF 转换组件的完整 Capture 安装。
- [Cadence 社区的 EDIF 导出步骤](https://community.cadence.com/cadence_technology_forums/pcb-design/f/allegro-x-capture-cis/58322/error-orcap-1153)
