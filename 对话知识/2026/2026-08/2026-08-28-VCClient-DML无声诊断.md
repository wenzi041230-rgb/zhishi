---
type: 对话知识
created: 2026-08-28 14:37
updated: 2026-08-28 15:10
source: Codex 对话
status: 已确认
tags:
  - 对话知识
  - VCClient
  - DirectML
  - 音频诊断
---

# VCClient DML 无声诊断与修复

## 用户目标

检查并修复一套 VCClient 2.1.4-alpha DirectML 安装没有声音输出的问题。

## 已确认结论

- 用户观察到的 `voice-changer-native-client.exe` 只是音频客户端，命令行指向 `http://localhost:18080/`；检查时该进程没有任何网络连接。
- 主程序首次启动从 14:16 开始下载约 2.5 GB 的 RVC 模块，直到 14:34 才监听 `0.0.0.0:18080`。音频客户端在服务器就绪前已经启动，并且没有自动重连。
- 当前服务器主页返回 HTTP 200，但本地音频接口报告 `local_voice_changer_interface_active=false`。
- 当前模型加载返回错误码 `701`：`rmvpe_20231006.onnx` 无法通过 ONNX Runtime 解析，错误为 `INVALID_PROTOBUF`。
- 损坏文件大小为 `115343360` 字节，实际 SHA-256 为 `66797e24367d60d28628ac3f525a6c072dfd027e14bc5d59db565864971f4f9f`，与程序声明的期望值 `84f0586308e36157f75b77c8591bf636d6719c0c4ba95f8faf3df479e7566219` 不一致。文件损坏或下载不完整是当前模型无法工作的直接原因。
- 当前音频路由本身具有明确含义：输入是 T7 GT 麦克风；主输出是 `CABLE Input (VB-Audio Virtual Cable)`；监听输出是 T7 GT 扬声器。主输出进入虚拟声卡，不会直接从物理扬声器播放。

## 修复结果

- 将原损坏 RMVPE 文件移入独立备份目录后重启，确认 VCClient 内置下载器在 HTTPS 连接中断时会留下半截文件，但仍继续启动服务器；第一次自动重下所得文件仍无法通过哈希和 ONNX 解析，不能以界面显示“完成”作为验收依据。
- 内置模块检查还发现 `applio_chinese_hubert_base.pt` 不完整。对两个文件使用支持断点续传、失败重试和分段并行的下载方式，且只在完整长度和程序声明 SHA-256 均一致后装回：
  - `rmvpe_20231006.onnx`：`362003174` 字节，SHA-256 `84f0586308e36157f75b77c8591bf636d6719c0c4ba95f8faf3df479e7566219`。
  - `applio_chinese_hubert_base.pt`：`1136482241` 字节，SHA-256 `8cd5db6302ae2e79b5972cd02ae375a42a76170374d6e1952fa78d1fe4e4f756`。
- 修复后模块 API 将全部 10 个模块报告为 `downloaded=true, valid=true`，服务器正常监听 `18080`，所选 RVC ONNX 模型和 RMVPE 管线成功加载，不再出现 `INVALID_PROTOBUF`。
- 本地音频接口已启动并报告 `active=true`；日志确认输入设备 3（T7 GT 麦克风）、主输出设备 6（VB-CABLE）和监听设备 5（T7 GT 扬声器）均以 44100 Hz 打开，并产生了转换/交叉淡化处理记录。
- 日志中的 CUDA 探测失败不阻断本次运行；当前模型实际使用 CPU/ONNX 执行路径。是否能听到实际说话仍需用户现场讲话验证，软件接口和日志不能代替物理听音。

## 决策与依据

- 未修改程序、配置、模型或音频设备，只使用进程、端口、日志、本地只读 API 和 SHA-256 证据诊断。
- 旧日志曾出现 `SlotInfo` 缺少 `chunk_sec` 的转换异常，但当前模型参数文件已含该字段，因此该问题属于上一轮运行的次要异常，不作为本轮无声的首要原因。

## 现场验收

- 实际讲话确认 T7 GT 监听端能听到变声，避免麦克风与扬声器距离过近产生啸叫。
- 聊天软件的麦克风选择 `CABLE Output (VB-Audio Virtual Cable)`，不要选择 `CABLE Input`。
- 若监听有声而聊天软件无声，故障边界已转移到聊天软件的输入设备选择或权限，不应再次重下模型。

## 可复用知识

- VCClient 的原生音频客户端进程存在不代表它已连上服务器；应同时检查目标端口监听和客户端连接。
- ONNX Runtime 的 `INVALID_PROTOBUF` 配合文件哈希不一致，可直接判定模型模块文件损坏，不应继续调整音量或声卡设置掩盖根因。
- 大文件下载遇到 `IncompleteRead`、`ChunkedEncodingError` 或 TLS 握手失败时，VCClient 2.1.4-alpha 的完成提示不可靠；必须同时核对文件长度、程序声明哈希和实际模型加载。
- VCClient 输出到 `CABLE Input` 时，变声结果应由其他软件从 `CABLE Output` 读取；用户直接听声依赖独立的监控设备。

## 关联知识

- [[对话知识/2026/2026-08/2026-08-28-VCClient安装与启动验证|VCClient 安装与启动验证]]
