---
type: 对话知识
created: 2026-08-28 14:37
updated: 2026-08-28 14:37
source: Codex 对话
status: 已确认
tags:
  - 对话知识
  - VCClient
  - DirectML
  - 音频诊断
---

# VCClient DML 无声诊断

## 用户目标

只读检查一套 VCClient 2.1.4-alpha DirectML 安装为何没有声音输出。

## 已确认结论

- 用户观察到的 `voice-changer-native-client.exe` 只是音频客户端，命令行指向 `http://localhost:18080/`；检查时该进程没有任何网络连接。
- 主程序首次启动从 14:16 开始下载约 2.5 GB 的 RVC 模块，直到 14:34 才监听 `0.0.0.0:18080`。音频客户端在服务器就绪前已经启动，并且没有自动重连。
- 当前服务器主页返回 HTTP 200，但本地音频接口报告 `local_voice_changer_interface_active=false`。
- 当前模型加载返回错误码 `701`：`rmvpe_20231006.onnx` 无法通过 ONNX Runtime 解析，错误为 `INVALID_PROTOBUF`。
- 损坏文件大小为 `115343360` 字节，实际 SHA-256 为 `66797e24367d60d28628ac3f525a6c072dfd027e14bc5d59db565864971f4f9f`，与程序声明的期望值 `84f0586308e36157f75b77c8591bf636d6719c0c4ba95f8faf3df479e7566219` 不一致。文件损坏或下载不完整是当前模型无法工作的直接原因。
- 当前音频路由本身具有明确含义：输入是 T7 GT 麦克风；主输出是 `CABLE Input (VB-Audio Virtual Cable)`；监听输出是 T7 GT 扬声器。主输出进入虚拟声卡，不会直接从物理扬声器播放。

## 决策与依据

- 未修改程序、配置、模型或音频设备，只使用进程、端口、日志、本地只读 API 和 SHA-256 证据诊断。
- 旧日志曾出现 `SlotInfo` 缺少 `chunk_sec` 的转换异常，但当前模型参数文件已含该字段，因此该问题属于上一轮运行的次要异常，不作为本轮无声的首要原因。

## 待处理

- 关闭 VCClient 后，仅移走或删除损坏的 `modules/rmvpe/rmvpe_20231006.onnx`，再启动程序让它重新下载，并重新校验 SHA-256。
- 服务器确认监听 18080 后，再重新启动音频客户端或在界面中重新按“开始”。
- 若需要自己听到变声，启用并选择物理耳机作为“监控”；若供聊天软件使用，则聊天软件的麦克风应选择 `CABLE Output (VB-Audio Virtual Cable)`。
- 修复后仍需实际讲话，分别验证输入电平、变声音频、物理监听和虚拟声卡下游，HTTP 200 本身不能证明音频链路可用。

## 可复用知识

- VCClient 的原生音频客户端进程存在不代表它已连上服务器；应同时检查目标端口监听和客户端连接。
- ONNX Runtime 的 `INVALID_PROTOBUF` 配合文件哈希不一致，可直接判定模型模块文件损坏，不应继续调整音量或声卡设置掩盖根因。
- VCClient 输出到 `CABLE Input` 时，变声结果应由其他软件从 `CABLE Output` 读取；用户直接听声依赖独立的监控设备。

## 关联知识

- [[对话知识/2026/2026-08/2026-08-28-VCClient安装与启动验证|VCClient 安装与启动验证]]
