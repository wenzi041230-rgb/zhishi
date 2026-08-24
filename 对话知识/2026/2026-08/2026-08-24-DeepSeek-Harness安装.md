---
type: 对话知识
created: 2026-08-24 11:40
updated: 2026-08-24 12:35
source: Codex 对话
status: 已确认
tags:
  - 对话知识
  - DeepSeek
  - Harness
  - dsh
---

# DeepSeek Harness 安装

## 用户目标

在 Windows 本机安装 DeepSeek Harness。

## 已确认结论

- 官方项目为 DeepSeek AI 的开源 `deepseek-harness`，命令行入口为 `dsh`。
- 已通过 npm 全局安装 `@deepseek-ai/dsh@0.1.1-rc.2`。
- `dsh --version` 返回 `0.1.1-rc.2`，`dsh --help` 可正常显示。
- 使用 `dsh web --no-open --host 127.0.0.1 --port 3080` 启动后，本地首页返回 HTTP 200；验证进程已关闭。
- 浏览器出现 `ERR_CONNECTION_REFUSED` 时，检查确认 3080 端口没有监听；重新启动后台 Web 服务后恢复，首页再次返回 HTTP 200。
- 已在桌面创建可双击启动的批处理文件，实测能启动 Web UI 并打开默认浏览器。

## 决策与依据

- 采用官方推荐的 npm 入口，避免直接维护源码构建树。
- 当前版本仍是 developer preview，官方提示可能有兼容性破坏变更。
- 未配置或写入 API Key，也未发起模型请求。

## 操作与产物

- 全局 npm CLI：`C:\Users\CLX\AppData\Roaming\npm\dsh.ps1`
- 启动方式：`dsh web`
- 后台启动方式：`dsh web --no-open --host 127.0.0.1 --port 3080`
- 桌面启动文件：`C:\Users\CLX\Desktop\DeepSeek Harness.bat`
- 默认本地地址：`http://127.0.0.1:3080`

## 待确认或待实测

- 需要在 Harness 设置中配置 DeepSeek API Key 后，才能进行实际模型对话。
- 若要长期运行，需另行决定工作区范围和权限模式；官方建议限制本地权限并审查 Agent 执行的代码。

## 可复用知识

- 安装前检查 Node.js；本机 Node.js 24.16.0 可运行该 CLI。
- 基础页面验证不等于模型服务、工具调用或生产环境安全性验证。
- `ERR_CONNECTION_REFUSED` 在本机首先应检查 `dsh` 进程和 3080 监听状态；服务退出后浏览器不会自动拉起 Harness。
- Windows `.bat` 文件应使用 CRLF 换行；Unix 换行可能导致 `cmd.exe` 拼接并错误解析命令。

## 关联知识

- 官方主页：https://www.deepseek.com/harness/
- 官方仓库：https://github.com/deepseek-ai/deepseek-harness
