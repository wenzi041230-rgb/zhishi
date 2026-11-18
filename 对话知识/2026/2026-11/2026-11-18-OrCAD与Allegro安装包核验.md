---
type: 对话知识
created: 2026-11-18 20:52
updated: 2026-11-18 20:52
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
