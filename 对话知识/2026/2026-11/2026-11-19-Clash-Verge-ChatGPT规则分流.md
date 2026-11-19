---
type: 对话知识
created: 2026-11-19 11:17
updated: 2026-11-19 11:17
source: Codex 对话
status: 已确认
tags:
  - 对话知识
  - 网络分流
---

# Clash Verge 的 ChatGPT 规则分流

## 用户目标

使用规则模式访问 ChatGPT 网页和 Windows 桌面端，让其他应用按订阅已有规则分流。

## 已确认结论

- 在 Clash Verge Rev 2.4.4 和 Mihomo 1.19.17 的现场配置中，全局模式与规则默认组选择了不同节点。原订阅缺少 ChatGPT 专用规则，规则默认组的节点显示连接错误。
- 在当前订阅的扩展脚本中创建独立选择组，优先使用全局模式下已验证的节点。将 chatgpt.com、openai.com、oaistatic.com、oaiusercontent.com、oaistatsig.com、openaimerge.com 的域名后缀规则以及 ChatGPT.exe 的进程规则置于原有规则之前。
- 前置域名规则涵盖浏览器访问相关域名的流量。进程规则能捕获桌面应用发起且由 Mihomo 识别的第三方连接。订阅原有规则保留了顺序和内容。
- UI 切换后的内核运行模式与启动配置一度不同。将本地基础配置的 mode 保存为 rule，正常退出并重新启动后，核实内核、启动配置和生成配置均为规则模式。

## 操作与产物

- 扩展脚本绑定在当前订阅，改动前的脚本已本地备份。专用组在 Clash Verge 的代理页可单独切换节点。
- 重启后通过本机连接表与 Mihomo 连接记录关联，确认 ChatGPT.exe 的真实连接命中域名规则和进程规则并收到数据。ChatGPT HTTPS 探测返回 200，OpenAI API 未认证探测按预期返回 401，百度匹配 DIRECT 且 HTTPS 返回 200。

## 待确认或待实测

- 重新登录、上传、语音及切换到其他订阅的完整交互未实测。忽略系统代理或绕开 Mihomo 的连接无法由该规则捕获。

## 可复用知识

- 先比较全局组和规则默认组各自选择的实际节点，再检查 ChatGPT 域名命中的规则与最终代理链。
- 测试浏览器域名分流与桌面进程分流时，分别检查真实连接和对照网站。HTTPS 探测可证明连接到达相应服务，功能交互需要单独验收。
- [OpenAI 网络要求](https://help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps)
- [Mihomo 路由规则](https://wiki.metacubex.one/en/config/rules/)
- [Clash Verge Rev 订阅扩展](https://www.clashverge.dev/guide/extend.html)
