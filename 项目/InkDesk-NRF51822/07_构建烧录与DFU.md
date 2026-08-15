---
title: InkDesk 构建烧录与 DFU
tags:
  - project/inkdesk
  - build
  - jlink
  - dfu
updated: 2026-08-15
---

# 构建、烧录与 DFU

## 工具链

- nRF5 SDK 12.3.0
- ARM GCC 9-2019-q4-major
- GNU Make 4.4.1
- SEGGER J-Link V8.18
- Nordic pc-nrfutil 6.1.7，用于 nRF5 Secure DFU 包和 settings

## 构建应用

```powershell
$env:PATH='E:\NRF51822Toolchain\tools\xpack-windows-build-tools-4.4.1-3\bin;E:\NRF51822Toolchain\tools\arm-none-eabi-gcc-9-2019-q4-major\bin;'+$env:PATH
Set-Location E:\NRF51822Build\projects\epd_ble_xfer
make clean
make -j4
```

构建成功不能证明实板 BLE、5 V 和屏幕都正常，仍需执行 [[08_故障排查与最短验证路径|实板验证]]。

## J-Link 两种方式

### App-only

适合板上已经存在正确 S130 和 Secure Bootloader 的情况。写入：

1. 应用 HEX。
2. 与该应用版本、大小和 CRC 匹配的 settings HEX。

### Full flash

适合首次烧录、芯片已擦除或 Bootloader 救援。先整片擦除，再依次写入：

1. S130 2.0.1。
2. Secure Bootloader。
3. InkDesk App。
4. Bootloader settings。

Full flash 是破坏性操作，不能用于需要保留其他数据的板卡。

## Secure DFU

- App 命令 `0x08` 设置 GPREGRET 后复位。
- Bootloader 以 `DfuTarg` 广播。
- v1.0.0 OTA 包 application version 为 6，目标 hw-version 51，SoftDevice requirement 为 `0x87`。
- 升级包使用 ECDSA P-256/SHA-256 签名。
- OTA 应在近距离、稳定供电下进行；当前硬件不把远距离 OTA 作为正式卖点。

## 密钥规则

- 私钥只用于本地签名，不进入 Obsidian、Git、源码 ZIP 或日志。
- Git 仓库忽略 `*.pem` 和 `*.key`。
- GitHub 只同步知识笔记，不同步烧录二进制和签名私钥。

## 2026-08-15 实板烧录记录

- 新收到的交接压缩包与此前交接源码逐文件一致，不是新固件版本；差异仅为旧分析目录中额外存在的本地晶振测试编译产物。
- 产品固件 v1.0.0 重新干净构建成功，应用大小 24,524 字节，HEX 与既有交付物一致。
- J-Link 识别到 NRF51822 XXAC / Cortex-M0，VTref 为 3.300 V；板上 S130、应用区和安全 Bootloader 均存在有效向量。
- 采用 App-only 方式写入应用与 v6 settings，保留 S130 和 Bootloader；J-Link 编程与 Verify 均成功。
- 从 `0x0001B000` 精确回读 `0x5FCC` 字节，SHA-256 与构建 BIN 一致：`EAE536A88B661FC42A4214BE98D3D433089FDBEC43D9E839B4C41D670800985E`。
- settings 位于 `0x0003FC00`，记录 application version 6、application size `0x5FCC`、有效 App 标记。
- 这证明目标芯片上的应用字节与构建产物一致，但不等于 BLE 空口、屏幕 5 V 或实际刷新已通过。
- 再次执行 v1.0.0 App-only 流程时，J-Link 报告应用和 settings 已与板上内容相同并跳过写入；复位、回读仍通过，但 Windows 20 秒扫描仍未发现 `EPD_Ink`。重复烧录同一映像不能作为新的 BLE 变量。

关联：[[03_固件架构与内存布局]]、[[10_交付物与安全边界]]。
