---
type: 对话知识
created: 2026-11-18 20:52
updated: 2026-11-18 21:09
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
- 已实测新建测试原理图、放置导线和将 DSN 写入磁盘。启动程序及测试文件留在目标数据盘。开始菜单入口首次验证能启动 Capture 并到产品选择框，但尚未完成重启后重新打开 DSN 的验证。
- 新建时日志提示 CAP0036，精简包缺少 CAPSYM.OLB 和常用符号库。需要用兼容的 OLB 补齐元件来源。Demo Edition 有设计规模限制，复杂工程保存未测试。
