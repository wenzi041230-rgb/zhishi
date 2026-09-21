---
type: 对话知识
created: 2026-09-20 16:30
updated: 2026-09-21 13:09
source: Codex 对话
status: 部分确认
tags:
  - 对话知识
  - GS Capture Studio
  - GSX
  - Gaussian
---

# GS Capture Studio V1.0 MVP 实际开发启动

## 用户目标

从高斯泼溅 APP 方案阶段切换到持续构建，建立真实工程结构、拆分任务、创建代码文件并实现可验收的 V1.0 MVP 核心模块。

## 已确认结论

- 已在项目输出目录建立 `GS-Capture-Studio` Python 工程，版本暂定 `0.1.0`。
- 当前 MVP 闭环为：GSX ZIP → 安全校验 → 项目准备 → COLMAP 探测 → 合成或外部 Gaussian 后端 → `scene.ply` + `report.json`。
- 默认 synthetic 后端是离线验证路径，不代表真实照片重建或 Gaussian 训练。
- GPT-5.6-sol 高思考独立评审给出 Conditional Go：可以推进离线核心工程，但真实 COLMAP/Gaussian 仍必须单独验收。

## 决策与依据

- 先做核心引擎和 GSX，不先做 Android、Qt、Viewer 或云服务；依据是需要先冻结可测试的数据协议和重建编排边界。
- 对外明确区分 `synthetic`、`auto`、`real` 三种后端模式；`auto` 降级必须进入报告，`real` 缺工具或失败不得静默降级。
- 报告和合成点云不写入时间戳、临时目录或绝对路径，以支持重复运行的字节级确定性。
- ZIP 校验包含路径穿越、绝对路径、重复路径归一化碰撞、符号链接、成员数量、解压总量和压缩比限制。

## 操作与产物

- 工程结构包含 `engine/`、`format/`、`scripts/`、`tests/`、`docs/` 和 `samples/`。
- GSX Reader/Validator 支持 manifest、camera、poses、图像引用、IMU warning 和 SHA-256 校验。
- COLMAP 适配器已实现工具探测和 feature extractor / exhaustive matcher / mapper 编排；本机当前未找到 COLMAP。
- Synthetic Gaussian backend 可生成带明确 `result_kind synthetic` 标记的 ASCII PLY，并进行 PLY 产物校验。
- 已生成可复现的 `samples/demo_room.gsx`；CLI 已验证 `validate`、`generate` 和 `inspect`。
- 测试结果：37 个 unittest 全部通过；重复运行 GSX、`scene.ply`、`report.json` 哈希一致。

## 待确认或待实测

- 指定版本的 COLMAP、CUDA、PyTorch 和具体 Gaussian trainer 尚未安装或冻结。
- 真实图片集上的相机位姿、稀疏重建、训练、模型质量、GPU 性能和失败诊断均未验收。
- Android ARCore Pose、PC Studio 和安装包属于后续里程碑；Camera2 连续帧/相机记录/IMU 原始采集骨架、GSX Builder 与无 GPU 预览器已完成代码构建范围。

## 可复用知识

- “代码存在”不等于“工具链已支持”；必须将软件闭环、外部工具运行、物理/视觉质量分别记录。
- MVP 验收至少应包含：输入包校验、严格/降级模式、确定性、产物独立校验、失败不覆盖上次成功结果。
- 下一阶段进入真实重建前，先冻结 GSX 1.0 子集、工具版本和受控数据集，再做端到端基线。

## 验收方法

- 先运行 `python -m unittest discover -s tests -v`，当前应为 37 个测试全部通过。
- 再运行 `python scripts/create_demo_gsx.py`、`python gs.py validate samples/demo_room.gsx`，应看到 `valid: true`、`frame_count: 3`。
- 运行 `python gs.py generate samples/demo_room.gsx --backend synthetic --output runtime`，检查 `report.json` 中 `result_kind: synthetic`、`reconstruction_performed: false`、`scene_ply_created: true`，并确认 `scene.ply` 含 `comment reconstruction_performed false`。
- 将同一输入生成到两个目录并比较 `scene.ply`、`report.json` SHA-256，验证确定性。
- 运行 `--require-real --gaussian-command "trainer"`，在未安装 COLMAP 时应返回退出码 4；这证明严格模式不会静默回退。
- 以上通过只代表软件闭环和安全边界通过，不代表真实 COLMAP/Gaussian、GPU 性能或视觉质量通过。
- 当前项目白话定义：最终要做的是“手机拍一圈现实物体或房间，电脑自动生成可旋转的 3D Gaussian 场景”；当前代码只是这套系统的电脑端引擎测试骨架，还不是手机 APP、桌面 UI 或完整产品。

## 本轮收口

- 新增 `scripts/verify_mvp.py` 一键离线验收，输出 `OFFLINE MVP: PASS` 和 `REAL RECONSTRUCTION: NOT TESTED`。
- 增加 auto fallback 契约、外部工具非零退出/超时/坏产物测试，以及旧成功场景和报告不被失败运行覆盖的测试。
- 固定退出码：0 成功，2 GSX 无效，4 严格真实模式缺工具或配置，5 外部工具失败/超时，6 外部工具产物无效。
- 外部工具调用不下载依赖；当前 Python 3.12.10 可用，COLMAP 仍未安装。

## 真实重建环境审计

- 2026-09-20 只读检查确认：Python 3.12.10 可用；COLMAP、`nvidia-smi`、`nvcc`、Conda、uv、`ns-train` 均未找到；torch、gsplat、nerfstudio 均未安装。
- 当前主机可以继续运行离线 MVP，但不能直接进行真实 COLMAP/Gaussian 重建；下一步应先选择目标机器、固定一个 Trainer 和匹配的运行时，再实施单一适配器。
- 已新增 `docs/ENVIRONMENT_AUDIT.md`，没有擅自安装重型依赖。

## 非 GPU MVP 独立验收修复

- 独立验收发现并修复四类可信度问题：Builder 不再为缺失 Pose ID 自动补序号；对象形式的 Pose 输入若坐标系缺失或与 GSX 1.0 冲突会被拒绝；Viewer 只有在 `report.json` 内记录的 `scene.ply` SHA-256 与当前文件一致时才信任 synthetic/real 来源标识；相机字段补齐类型和有限数校验。
- 相机 `width`、`height` 现在要求为正整数，`fps` 和 `intrinsic.fx/fy/cx/cy` 要求为非布尔有限数，且 `fx/fy` 必须为正数。
- 增加缺失 ID、坐标系冲突、相机/Pose 非有限数、重复 ID、失败重建保留旧有效包、Viewer 报告哈希不匹配和 `reconstruction_performed=false` 等回归测试。
- 复核追加发现并修复两个边界：超大整数转浮点时捕获 `OverflowError` 并正常报告无效；scene/report 改为成对暂存、备份和回滚，报告暂存失败不会留下新场景与旧报告的混合状态。
- 2026-09-20 验证结果：37 个 unittest 全部通过；一键验收输出 `OFFLINE MVP: PASS` 和 `REAL RECONSTRUCTION: NOT TESTED`，合成 `scene.ply` SHA-256 为 `7926b24bfeeb4a7ad0cb29992030bba9177bb19e58c09a6c6489b2a84a5c36c3`；缺少 COLMAP 时 strict real gate 正确拒绝；`compileall` 通过。
- 证据边界保持不变：以上只证明非 GPU 离线软件闭环和输入/来源防误标边界，不证明真实 COLMAP、Gaussian 训练、视觉质量或 GPU 性能。
- 2026-09-21 重新运行 37 个测试、一键离线验收和 `compileall`，结果保持通过。最后一次 GPT-5.6-sol 高思考独立验收因账户用量限制未完成，未将其计入验收证据；非 GPU 结论仍以本地测试和验收脚本为准，真实重建仍待后续环境验收。

## Android 非 GPU 骨架

- 新增 `mobile/` Android 工程，使用本机 JDK 17、Gradle 和 Android SDK 36 构建。
- 已从“导入照片测试入口”改为 Camera2 后置摄像头事实采集：实时预览并连续保存 JPEG 帧，不再把文件导入当作产品采集流程。
- 每个采集会话同步记录相机帧的 sensor timestamp、相机 ID/分辨率/帧率/内参状态，以及加速度计和陀螺仪 `imu.csv`；结束状态明确写入 `pose_available=false`。
- 新增 `GsxCaptureWriter`：要求每帧真实整数 ID、时间戳、位置、非零四元数和固定坐标系；校验图片、相机参数、重复 ID、时间单调性后生成带 SHA-256 的 GSX，并使用临时文件替换。
- `gradle assembleDebug` 已通过，APK 已确认包含 CAMERA 权限和主 Activity。
- 2026-09-21 在真实小米 2210132C / Android 16 设备上完成 ADB 安装、启动和摄像头采集实测：授予 CAMERA 后连续运行并停止保存，生成 594 个 JPEG 帧、594 条 `frames.csv` 帧记录、约 2102 条 IMU 数据行、`camera.json`（1920x1080、30fps、内参来自 CameraCharacteristics）和 `capture_status.json`；应用界面确认会话已保存，未观察到崩溃。
- 对实机会话做了离线一致性复核：`frames.csv` 为 1 行表头 + 594 行数据，JPEG 文件为 594 个；所有引用路径均存在且无多余帧，帧 ID 连续，Camera sensor timestamp 唯一且严格递增，JPEG 文件均非零长度。
- 实机证据只确认“真实 Camera2 流 + 相机时间戳 + IMU 原始记录 + 本地会话落盘”可用；`pose_available=false` 且状态为 `UNAVAILABLE_ARCORE_NOT_CONNECTED`，所以仍不能导出有效 GSX，也没有证明真实 COLMAP/Gaussian 重建。
- GPT-5.6-sol 高思考独立验收结论：原始相机帧 + IMU 子功能通过实机冒烟验收；完整 Android 摄像头采集 MVP 不通过，P0 原因是没有逐帧真实相机位姿。下一项最高价值工作是接入 ARCore Pose 并完成帧、Pose、IMU 的时间关联和 GSX 严格校验。
- ARCore Pose 适配器仍未接入，当前原始摄像头流不是可直接导出的有效 GSX；不能把当前 Android 骨架称为完整手机采集 APP。

## 关联知识

- [[对话知识/00_对话知识索引]]
