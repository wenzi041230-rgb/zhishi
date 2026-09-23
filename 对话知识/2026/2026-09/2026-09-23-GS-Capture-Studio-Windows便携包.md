---
type: 对话知识
created: 2026-09-23 15:09
updated: 2026-09-23 15:09
source: Codex 对话
status: 部分确认
tags:
  - 对话知识
  - GS-Capture-Studio
---

# GS Capture Studio Windows x64 便携包构建与静态验收

## 用户目标

继续完成 GS Capture Studio 的非 GPU 工程工作；需要重要决策和验收调用 GPT-5.6-sol 高思考，真实验证等用户接入后再进行。

## 已确认结论

- Windows 11 x64 上使用 Python 3.12.10 与 PyInstaller 6.20.0 成功生成 one-folder/windowed 桌面监视器便携包。
- 目录包大小 27,899,024 字节；ZIP 为 11,996,935 字节、998 个条目；SHA-256 `A2B78D0BEF150C8ADECCA422F9A294B141D2AAFDFF1FFAAAE0F8EC5027984571` 与旁侧校验文件一致。
- 包内包含项目、Python、Tcl/Tk、PyInstaller 与 OpenSSL 许可文本及构建信息；静态清单中未发现 runtime、手机采集会话、GSX/JPEG 样例、tests、samples 或 mobile 目录。EXE 未签名。
- 全量 Python 单测 79/79 通过，`compileall` 通过；程序运行时和真机尚未验收。

## 决策与依据

- 首包选择 Windows x64 便携目录 ZIP，不做单文件 EXE 或安装器；便于检查、诊断，并避免对系统安装状态造成影响。
- ADB 保持外置，要求用户另行安装 Platform-Tools 并加入 PATH；程序只在用户点击开始监视时才访问 ADB。
- 冻结版数据默认写入 Windows `Personal` Known Folder 下的“GS Capture Studio”；找不到目录或目录不可写时要求用户选择，不能回退到程序包目录。
- `.obsidian/graph.json` 是既有用户改动，知识记录提交未包含该文件。

## 操作与产物

- 构建脚本：`outputs/GS-Capture-Studio/scripts/package_windows.ps1`
- 便携 ZIP：`outputs/GS-Capture-Studio/dist/GS-Capture-Studio-Windows-x64.zip`
- SHA-256：`outputs/GS-Capture-Studio/dist/GS-Capture-Studio-Windows-x64.zip.sha256`
- 使用说明：`outputs/GS-Capture-Studio/docs/WINDOWS_PORTABLE.md`
- 代码目录目前不是 Git 仓库；知识笔记与代码工作区独立管理。

## 待确认或待实测

- 本轮没有启动 EXE，也没有在无 Python 的环境解压运行；中文/空格路径、只读程序目录、ADB 缺失/无设备、真机导入与窗口安全退出均待实测。
- ADB 当前没有连接设备；需要用户接入后完成 UI 和真机测试。
- EXE 未签名，当前只适合内部验证；面向外部发布前仍需决定签名、安装器及最低 Windows 版本。
- 虽通过参数请求了 GPT-5.6-sol/high 复核，但复核执行方不能确认模型身份且一度引用旧包哈希；针对最新哈希的第二次请求未返回，故没有将其意见冒充指定模型最终验收。

## 可复用知识

- PyInstaller 的 `.gitignore` 规则不会自动控制打包输入；必须从明确的入口构建，并审计最终 ZIP，不要把运行目录、测试数据或截图带入。
- 程序目录与用户数据目录应分离；Frozen 应用不得把可变状态写到 `_internal`、安装目录或 onefile 临时目录。
- “构建成功 + 静态 ZIP 检查”不等同于 EXE 已启动或设备功能已验收；路线图需保留目标机器人工验证项。

## 关联知识

- [[对话知识/2026/2026-09/2026-09-20-GS-Capture-Studio-MVP|GS Capture Studio MVP 与桌面导入验收]]
- [[对话知识/2026/2026-09/2026-09-23-GS-Capture-Studio-桌面监视器UI|桌面监视器窗口 MVP 验收]]
- [[对话知识/00_对话知识索引]]
