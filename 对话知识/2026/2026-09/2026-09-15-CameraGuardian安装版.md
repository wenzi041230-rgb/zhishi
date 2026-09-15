---
type: 对话知识
created: 2026-09-15 11:15
updated: 2026-09-15 11:15
source: Codex 对话
status: 部分确认
tags:
  - 对话知识
  - CameraGuardian
  - Windows安装程序
  - InnoSetup
---

# CameraGuardian Windows 安装版

## 用户目标

把已验证的 CameraGuardian 精准低延迟便携版制作成可直接运行、可升级和可卸载的标准 Windows 安装程序。

## 已确认结论

- 使用 Inno Setup 6.7.3 生成 64 位兼容安装程序，版本为 1.0.0。
- 默认按当前用户安装到 `%LOCALAPPDATA%\Programs\CameraGuardian`，不需要管理员权限。
- 默认创建开始菜单和桌面快捷方式；“登录 Windows 后自动启动”可选且默认关闭。
- 安装结束可选启动程序；Windows“已安装的应用”中提供卸载入口。
- 卸载保留 `%LOCALAPPDATA%\CameraGuardian\settings.json`，重装或升级可沿用摄像头编号和界面开关。
- 复用既有 PyInstaller onedir 成品，`_internal`、两个本地视觉模型、原生库和中文说明全部随安装包部署。

## 决策与依据

- 采用每用户安装，避免为了摄像头工具引入不必要的管理员授权。
- 开机启动不默认开启，避免摄像头分析在用户不知情时随登录自动运行。
- 使用固定 AppId `{85A5CB74-1C76-492A-969E-0F36574C617F}`，后续相同 AppId 和更高版本号可覆盖升级。
- 简体中文安装界面使用 MIT 许可的 `kira-96/Inno-Setup-Chinese-Simplified-Translation`，翻译文件和许可证副本随构建脚本保存。

## 操作与产物

- 新增 `installer/CameraGuardian.iss`、`build-installer.ps1`、中文语言文件和第三方许可说明。
- 重建核心程序前执行 `93 passed`，`pip check` 返回无依赖冲突。
- 安装器已在项目内隔离目录完成“静默安装 → 无摄像头启动 → 静默卸载”闭环；退出码均为 0，卸载后程序文件和测试目录均不存在。
- 安装程序：`E:\CodexApps\CameraGuardian\release\CameraGuardian-Setup-x64.exe`
- 文件版本：`1.0.0.0`；产品版本：`1.0.0`
- 大小：80,563,290 字节
- SHA256：`4166902B13650C25D883FAA116A38951A1738EFB6A70861F342AD939BF5FB645`

## 待确认或待实测

- 安装程序没有商业代码签名证书，Windows SmartScreen 可能显示“未知发布者”；SHA256 可用于传输后完整性核对。
- 本机 Microsoft Defender 服务处于关闭状态，单文件扫描接口不可用，不能声称安装包已通过 Defender 扫描。
- 隔离验证没有创建真实桌面和开始菜单快捷方式，也没有再次占用摄像头；这些入口已通过 Inno Setup 编译，实际交互和安装路径摄像头运行仍可在首次正式安装时确认。

## 可复用知识

1. 大型 PyInstaller onedir 应由安装器递归部署整个目录，不能只安装主 EXE。
2. 构建安装器前先验证依赖、自动测试、模型和原生库，再做静默安装/启动/卸载闭环。
3. 摄像头应用的开机启动应是明确可选项，默认关闭更符合隐私预期。
4. 无可信证书时应明确标注未签名，不用自签名证书冒充受信任发布者。

## 关联知识

- [[项目/CameraGuardian摄像头守护]]
- [[对话知识/2026/2026-09/2026-09-09-CameraGuardian准确动作判定优化]]
- [[对话知识/2026/2026-09/2026-09-09-CameraGuardian切换WPS联动]]
