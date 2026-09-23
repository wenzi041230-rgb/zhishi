---
type: 对话知识
created: 2026-09-20 16:30
updated: 2026-09-23 10:38
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
- ARCore SharedCamera/Pose 适配代码已接入并可完成 Android 构建，但尚未在解锁且安装 ARCore 运行时的手机上完成真实 Pose 采集；不能把当前 Android 骨架称为完整手机采集 APP。

## ARCore Pose 本轮严格验收

- 2026-09-21 重新执行 Android `assembleDebug`，结果为 `BUILD SUCCESSFUL`；仅证明 APK 可以构建。
- 重新执行 Python 契约测试，37/37 全部通过；仅证明 GSX/Builder/Pipeline/Viewer 的离线契约保持通过。
- ADB 显示采集 APP 进程和 `MainActivity` 仍存在，但窗口焦点是 `NotificationShade`，同时 `mDreamingLockscreen=true`、`isKeyguardShowing=true`；这只能证明进程未退出，不能证明相机预览、ARCore TRACKING 或 Pose 采集成功。
- `pm path com.google.ar.core`、普通包列表和包含卸载包的列表均未发现 `com.google.ar.core`，因此 ARCore 运行时未安装。
- 严格结论：Android 构建 OK，37 项 Python 契约测试 OK，ARCore Pose 端到端验收 NG。没有执行解锁后的真实采集，不得记为 OK。
- 解锁后的最小重测：确认 ARCore 包已安装并可用；前台启动 APP 且无崩溃；完成一次有位移和转动的真实采集；确认图像、Pose、IMU 均有数据并完成时间关联；确认 `pose_available=true`、`pose_complete=true`、无待匹配帧；生成 `capture.gsx` 后用 Python Validator 验证通过，并抽查 Pose 为有限数、四元数非零、时间戳严格递增、引用图片和 SHA-256 一致。

## ARCore Pose 真机重测与 MVP 放行

- 设备为真实小米 2210132C / Android 16，ADB serial 为 `9ec40c76`；官方 ARCore 1.56.262080393 已通过手机系统安装器安装。安装包来源和校验记录：`Google_Play_Services_for_AR_1.56.0.apk`，SHA-256 `8C6DAC408FF9320515CE105FB17E3F76087762B6B7AD05B6DA49030FEE3839B6`。由于设备对 ADB 直接安装返回 `INSTALL_FAILED_USER_RESTRICTED`，没有绕过系统安全策略。
- 首次 ARCore SharedCamera 真机测试为 NG：相机有 3 张图像但没有 Pose，随后出现 MIUI camera pipeline force disconnect；确认不是“缺少 ARCore”而是额外 TextureView 预览面导致的共享流组合不兼容。
- 修复内容：按官方 SharedCamera 结构改为 ARCore 外部纹理作为可见预览、仅增加 CPU `ImageReader` 采集面；将 SurfaceTexture listener 延后到 ARCore resume 后；增加空值保护；将 `image_count` 定义为真正写入且已匹配 Pose 的图像数，并单独记录 `source_image_count`、`unmatched_image_count`。
- 最新真机会话 `session-1789969989290`：`image_count=73`、`frame_count=73`、`pose_frame_count=73`、`pose_available=true`、`pose_complete=true`、`pose_status=TRACKING`、`gsx_exported=true`；同时保存 `source_image_count=288`、`unmatched_image_count=215`、`imu.csv` 1015 条数据行。未匹配源图像不进入 GSX，不能宣称无丢帧或完整保存视频流。
- 真机端界面显示 OK，生成 `capture.gsx`，文件大小 4,363,802 bytes。主机侧 SHA-256 为 `8933B8AFAFA03319312D32EACA7713B34CCF67B0A205CA24591B9EAACDB4A9F6`；项目验证器返回 `valid=true`、`errors=[]`、`frame_count=73`、`checked_files=75`。
- 独立数据复核：73 条 `frames.csv` 数据行与 73 个 JPEG 一一对应；ID 连续；图像时间戳严格递增；最大图像/Pose 时间差 4,405,802 ns，小于 5 ms；位置、四元数均为有限数，最小四元数范数约 0.9999999；IMU 数据行非空。
- GPT-5.6-sol 高思考独立最终验收：**OK（范围限定）**。通过的是“实时摄像头 + ARCore Pose + GSX 导出 MVP”；`pending_image_count=120` 不阻止本次放行，因为交付物只包含 73 个成功配对帧，未匹配源图像已单独披露。剩余边界：当前 GSX 未打包 `imu.csv`，只证明会话目录保存了 IMU；COLMAP、Gaussian 训练、GPU 渲染和真实 Gaussian 重建仍未验收。

## 采集线程优化重测

- 为降低相机线程被 JPEG 压缩拖慢的问题，尝试过两种方案并按真机结果处理：保留 `Image` 对象到匹配队列会导致 ImageReader `acqCount` 达到上限并堵死相机，判定 NG；强制 15 FPS 会导致本机 ARCore 无法得到 Pose，判定 NG，均已撤回。
- 最终保留方案：恢复已验证的 30 FPS SharedCamera 参数；相机线程只复制 YUV 到 NV21，JPEG 压缩转移到独立编码线程；编码队列上限为 4，停止时等待编码队列完成后再写会话状态；不再持有未关闭的 `Image` 对象。
- 2026-09-21 真机重测会话 `session-1789970977358`：`image_count=137`、`source_image_count=437`、`unmatched_image_count=300`、`frame_count=137`、`pose_frame_count=137`、`pose_complete=true`、`pose_status=TRACKING`、`pending_image_count=120`、`imu.csv` 1521 条数据行、`gsx_exported=true`。手机界面显示 OK，采集期间无崩溃和 ImageReader 堵死。
- 主机侧 GSX SHA-256：`2255729F6DAB6ED32731F420DDA02E0425A752AF6AAF5911BFBD669B6DC105A9`；验证器返回 `valid=true`、`errors=[]`、`frame_count=137`、`checked_files=139`。数据复核通过：137 条 Pose 对齐记录、ID 连续、时间戳严格递增、最大 Pose 匹配误差 4,405,802 ns、小于 5 ms、数值有限、最小四元数范数约 0.9999999、IMU 非空。
- GPT-5.6-sol 高思考独立验收：**OK（有限放行）**。本轮只能宣称“摄像头 + ARCore Pose + IMU 原始记录 + 有效 GSX 导出”通过；不能宣称无丢帧或停止后所有图像均排空，437 个源图像中只有 137 个进入 GSX，`pending_image_count=120` 已披露为当前 MVP 非阻塞边界。GPU、COLMAP、Gaussian 训练和真实三维重建仍未验收。

## PC 接收与自动质量报告

- 新增 `scripts/import_android_capture.py`：通过 `adb exec-out run-as com.gscapture.mobile cat` 从 Android 私有目录接收指定或最新完整 `session-*`，先写入 `.partial` 目录，所有文件接收成功后再原子改名；ADB 失败会清理半成品，不覆盖已有会话。
- 新增 `engine/capture_report.py` 和 `tests/test_capture_report.py`：检查 Camera 参数、`frames.csv`、JPEG 引用与数量、Pose 有限数/四元数/时间差、IMU、状态字段、GSX Validator，并读取 `transfer_manifest.json` 逐文件验证存在性、字节数、SHA-256、重复路径、覆盖范围和必需文件。
- 返工前独立验收发现两个 P1：报告未验证传输清单，且未交叉核对 CSV/JPEG/状态/GSX 帧数；已补齐并增加哈希篡改、跨文件数量不一致测试。返工后 Python 测试为 42/42，`compileall` 通过。
- 使用真实小米会话 `session-1789970977358` 完成 PC 接收：传输清单与实际载荷均为 142 个文件，逐文件哈希/字节复核无错误；质量报告 `PASS`，`frames.csv`、JPEG、状态三项计数和 GSX `frame_count` 均为 137，IMU 为 1521 行，最大图像/Pose 时间差 4,405,802 ns，GSX `valid=true`、`errors=[]`、`checked_files=139`。
- 质量报告明确警告 437 个源图像中 300 个未匹配、120 个待处理；因此不能宣称无丢帧。GSX 本身仍未打包 IMU，原始 IMU 在会话目录 `records/imu.csv`；当前结果不代表 COLMAP、Gaussian 训练、GPU 渲染或真实三维重建。
- 第二次尝试重新接收时 ADB 设备瞬时消失，脚本正确失败且没有留下 `.partial`；已有真实接收会话经返工后的报告程序重新验证通过，不需要重新采集。
- GPT-5.6-sol 高思考第二次独立验收：**OK（限定范围放行）**；确认两个 P1 已修复，允许进入下一步非 GPU 工作。

## IMU 纳入 Android GSX

- GPT-5.6-sol 高思考决策：选择先补齐 Android `camera_stream` GSX 的 IMU 自包含能力，再做桌面自动监视；普通无 IMU GSX 保持兼容并继续 warning，`camera_stream` 或 `imu_required=true` 的包必须有 IMU。
- Python `engine/gsx.py` 新增 IMU 严格校验：精确表头、非空、正时间戳、有限数、仅允许 `accelerometer`/`gyroscope`、两类传感器均存在、各自时间戳严格递增、与图像 Pose 时间范围重叠；仅对 Android 采集类型强制存在。
- Android `GsxCaptureWriter` 新增带 IMU 文件的写包路径；`CaptureRecorder` 将已关闭的 `records/imu.csv` 原样写入 `imu/imu.csv`，并在 manifest 声明 `capture_kind=camera_stream`、`imu_required=true`，IMU 同步进入 SHA-256 checksums。GSX 规范补充兼容策略和校验要求。
- 回归测试扩展到 46/46；覆盖普通无 IMU 包兼容、Android 缺失 IMU、IMU 格式/传感器/时间边界、包内外 IMU 不一致和旧有报告门禁。`compileall`、`scripts/verify_mvp.py`（`OFFLINE MVP: PASS`、`REAL RECONSTRUCTION: NOT TESTED`）均通过。
- Android `gradle assembleDebug` 构建成功并安装到真实小米 2210132C。新真机会话 `session-1790040408549`：`image_count/frame_count/pose_frame_count=504`、504 个 JPEG、`pose_status=TRACKING`、`imu_rows=4203`（accelerometer 2102、gyroscope 2101）、`max_pose_delta_ns=2111851`、`pending_image_count=119`、`unmatched_image_count=718`。
- 新会话 PC 接收与报告：传输清单/实际文件 509/509，质量报告 `PASS`；GSX `valid=true`、`errors=[]`、`warnings=[]`、`frame_count=504`、`checked_files=507`。包内外 IMU 均为 280642 bytes，字节级一致，SHA-256 为 `0cb1f8c0052f79f5a78c88530547c8a11fd8896f2041563ba6ddca2bfbfc7120`。
- GPT-5.6-sol 高思考最终验收：**OK（范围限定）**。本轮只证明 IMU 已成为 GSX 自包含数据并通过严格校验；不证明相机与 IMU 已完成传感器融合，不证明无丢帧，也不代表 COLMAP、Gaussian、GPU 或真实三维重建已完成。718 个源图像未匹配、119 个停止时待处理，均已在报告中披露。
- 指标口径：`source_image_count` 是相机回调中观察到的源图像数，`image_count` 是成功写入且找到 ARCore Pose 的图像数；`unmatched_image_count` 是两者差值，可能包含编码队列满时未进入 JPEG、±5 ms 内找不到 Pose 或等待队列淘汰等多种原因，不能单独解释为某一种故障。`pending_image_count` 表示停止时仍在等待匹配的图像数。

## 未匹配帧分项诊断与第一轮修复

- 2026-09-22 对最新真实会话 `session-1790040408549` 做了原因分析：35.04 秒内 `source=1222`、`matched=504`，已保存帧中位间隔约 66.6ms（约 15 FPS），成功帧最大 Pose 时间差 2.11ms、p95 约 2.07ms，说明长期保存吞吐/队列策略比 ±5ms Pose 窗口更可疑。
- GPT-5.6-sol 高思考决策：不要先扩大队列或并行编码；长期吞吐不足时这只会延迟丢弃。保持 Camera2/ARCore 约 30FPS，只在 YUV/JPEG 之前按 sensor timestamp 选择约 15FPS；保持 JPEG quality 90、编码队列上限 4；如果仍过载，再以 JPEG quality 85 做单变量 A/B。此前真机历史还表明强制相机 15FPS 会导致 ARCore 无 Pose，因此本轮只对保存路径限流。
- Android `capture_status.json` 新增 `policy_selected_count`、`policy_skipped_count`、`selected_unmatched_image_count` 以及编码/队列/Pose 淘汰/时间戳间隔等计数；`unmatched_image_count` 继续保留源图像总差额，报告同时给出主动跳过和选中后失败，避免把主动策略误报成故障。
- `engine/capture_report.py` 新增分项账本和 Markdown 展示；旧会话仍兼容但会提示缺少诊断计数。新增回归测试覆盖分项计数报告。
- 第一轮修复已完成静态验收：Python unittest `47/47` 通过，`compileall` 通过，Android `gradle :app:assembleDebug` 构建成功，APK 已安装到真实小米 2210132C（ADB `9ec40c76`）。
- 真机重测尚未完成：设备当前处于锁屏，应用窗口被 `NotificationShade`/Keyguard 遮挡，不能安全开始摄像头采集；因此本轮不能宣称“未匹配问题已解决”。解锁后的验收门槛为：主动选中帧 `queue_full_drop=0`、编码/Pose失败均可解释、最终 `pending=0`、选中帧保存率至少 99%、分项账本 `unexplained=0`，并完成至少一次 35–60 秒真实采集后再决定是否做 JPEG quality 85 A/B。

## Pose 驱动时间关联修复与限定放行

- 2026-09-22 先对会话 `session-1790044735875` 做失败距离统计：未在 ±5 ms 内找到 Pose 的图像，其最近 Pose 距离最小约 5.022 ms、p95 约 39.220 ms、最大约 1.157 s。由此确认主要问题是固定 15 FPS 选择相位与 ARCore Pose 节奏错位，不是 JPEG 队列已被证明耗尽。
- GPT-5.6-sol 高思考决策：不扩大 ±5 ms 窗口，也不先做 Pose 插值；改为“Pose 驱动、直接 Pose、一对一”选择。每个相机候选图像先进入有界原始候选缓存，等可用的 ARCore TRACKING Pose 后在 ±5 ms 内匹配；匹配后只提交单一 JPEG 编码队列，图像对象立即关闭，避免 ImageReader 堵塞。
- Android 实现：`frame_selection_policy=pose_driven_direct_pose_match`，候选缓存上限 16，编码队列上限 4，JPEG quality 90；匹配 Pose 不复用、不插值、不伪造，停止时分别报告无 Pose、队列淘汰和编码失败。
- 最终真机验证会话 `session-1790045215982`：源图像 1186，直接匹配并写入 552，未匹配源图像 634；`policy_selected_count=552`、主动跳过 0、编码提交/完成均 552、队列丢弃 0、编码失败 0、待处理 0、Pose 队列淘汰 0、分项账本未解释数 0。成功帧约 38.3 秒，主机侧图像-Pose 最大时间差 1.388 ms、p95 1.091 ms。
- PC 接收与质量报告通过：557/557 文件传输和哈希校验通过；`frames.csv`、JPEG、状态、GSX 均为 552 帧；GSX `valid=true`、`errors=[]`；包内 IMU 与会话 IMU SHA-256 相同（`5a2d8f5d19aad27e952f5d35ff3393d5a4863e7321c0bfa28841c96f9edb05a3`）；IMU 共 4078 行。
- 本轮本地验收：Python unittest 47/47 通过，`compileall` 通过，Android `:app:assembleDebug` 成功；独立复核确认 Pose 数值有限、四元数范数约为 1、帧 ID 连续、时间戳递增，`capture.gsx` 验证通过。
- 放行边界：以上只证明“真实摄像头 + ARCore Pose 的时间关联与 GSX/PC 接收链路”在一次约 38 秒真机运行中通过，不能宣称 3 次重复、5 分钟稳定性、无源帧丢弃，也不能宣称 COLMAP、Gaussian、GPU 或真实三维重建已完成。最终 GPT-5.6-sol 独立验收因账户用量限制未完成，未将其伪装为验收证据；本次结论以真机、PC 报告和本地测试为依据。

## ARCore 同帧采集长测验收

- 为消除独立图像流与 Pose 流的会话相位不稳定，采集路径改为从同一个 ARCore `Frame` 调用 `Frame.acquireCameraImage()`，并使用该 Frame 的真实 Pose；图像复制完成后立即关闭，JPEG 继续进入有界编码队列。
- 三次 35–60 秒真机重复均通过：图像获取/保存分别为 605/605、605/605、600/600，最大图像-Pose 时间差分别为 0.656 ms、0.891 ms、1.119 ms；队列、pending、淘汰和输出空洞均为 0，GSX、IMU 和传输完整性均通过。
- 第一次约 5 分钟测试采集了 7627 帧，但旧版 GSX writer 在停止导出时将全部图片读入内存并发生 OOM；该轮判定 NG，不能用采集计数掩盖导出失败。writer 随后改为逐文件流式计算哈希和写 ZIP。
- 修复后约 5 分钟会话 `session-1790066449931`：eligible 4994、acquired 4993、not available 1、copy failure 0，图像/源帧/帧/Pose/JPEG/GSX 均为 4993；直接获取率 99.97998%，取得图像后的保存率 100%，最大图像-Pose 时间差 1.699598 ms，最大输出间隔 199.857708 ms。
- 该长测编码、队列、pending、Pose 淘汰、无 Pose 和未解释计数均为 0；Pose 为 `TRACKING` 且 complete，GSX 有效，包内外 IMU SHA-256 一致，传输清单有效，`gsx_exported=true`，停止和导出后应用进程仍存活。
- 独立验收结论：**PASS（范围限定）**。唯一一次 `direct_camera_image_not_available` 是已分类、可计量的暂时不可用，不是复制/编码/队列/导出失败；直接获取率仍显著高于 95% 门槛，也没有造成超过 250 ms 的输出空洞，因此 warning 不改变总体验收等级，不需要降为 conditional PASS。
- 放行范围仅为“真实手机摄像头 + 同帧 ARCore Pose + IMU + GSX 导出 + PC 接收完整性与约 5 分钟稳定性”。不能据此宣称 COLMAP、Gaussian 训练、GPU 渲染、视觉重建质量或完整产品 V1.0 已通过。

## 桌面端自动导入与会话管理架构决策

- 2026-09-23 决定采用方案 B：Python 长驻 watcher 轮询 ADB，并提供 `--once`；其实现形态必须是“可独立测试的单次扫描核心 + 很薄的循环”，而不是把业务规则写进无限循环。
- 现有 `scripts/import_android_capture.py` 继续作为单会话导入能力，负责 `.partial`、原子完成目录、`transfer_manifest.json` 和质量报告；watcher 只负责发现完整会话、按设备和会话去重、串行调用导入、汇总状态和在 ADB 恢复后继续工作。
- 文件系统是事实来源：最终会话目录表示已导入，既有 `.partial` 表示需要人工检查，watcher 不覆盖、不删除、不自动续传；`monitor_status.json` 仅是可重建的运行视图，必须原子写入，不能作为唯一数据库。
- 建议落盘为 `runtime/imported_captures/<safe-serial>/session-*`，避免多设备同名会话碰撞；同一输出根目录只允许一个 monitor 实例，使用 Windows 进程级文件锁防止并发重复导入。
- 质量 `PASS` 和 `FAIL` 都是已完成导入的终态：FAIL 必须保留报告并停止自动重试；无设备、ADB 暂时断开和没有新会话是可恢复状态，不得使长驻进程崩溃。
- MVP 不包含桌面 GUI、SQLite、Windows 服务/安装器、任务计划自动配置、并行多设备导入、断点续传、自动删除手机数据、自动清理 `.partial`、COLMAP、Gaussian、GPU 或云同步。
- 最小验收包括：重复扫描不重复导入；既有 `.partial` 零修改并报告阻塞；PASS/FAIL 正确落盘；无设备不崩溃；ADB 中断后重连可继续；状态文件始终为有效 JSON；两实例并发被锁拒绝；所有自动测试通过，并完成一次 Windows `--once` 假 ADB 隔离测试。

## 桌面端自动导入 MVP 独立验收

- 2026-09-23 只读独立验收结论为 **NG**。通过项包括：Python unittest 54/54、`compileall`、16 个真实会话导入、16 份质量报告、12 PASS/4 FAIL 如实保留、传输清单完整性全部有效、第二次扫描不重复复制、缺失 `capture.gsx` 会话保持 `WAITING_COMPLETE`、单实例锁实测拒绝第二个 watcher。
- 阻断项一：`import_session` 先将 `.partial` 原子改名为正式目录，随后 watcher 才生成质量报告。故障注入证明，若报告构建或写入发生本地 `OSError`，第一次扫描返回 `IMPORT_ERROR`，但正式目录已存在且没有报告；第二次扫描会直接报告 `ALREADY_IMPORTED`，该会话永久不再补报告。这违反“正式目录代表已完成导入”的事实源约定。
- 阻断项二：本地 `OSError` 只在导入/报告阶段被归为 `IMPORT_ERROR`；设备选择、会话枚举和远端文件清单阶段仍将 `OSError` 归为 `WAITING_FOR_DEVICE`。隔离状态文件中的 `WinError 2` 实际表示本地程序不存在，却被写成 `WAITING_FOR_DEVICE`；定向故障注入可稳定复现。
- 现有 `test_local_storage_error_is_not_reported_as_device_loss` 只覆盖 `import_session` 抛出 `OSError`，未覆盖设备选择、会话枚举、远端清单，也未覆盖“正式目录已生成但质量报告失败”的第二次扫描行为。
- 放行前最低修复门槛：正式目录必须只在传输清单和质量报告都写成后出现，或扫描时把缺少报告的正式目录识别为明确错误并支持安全恢复；所有本地 `OSError` 必须与 ADB/设备断开分开归类；增加上述两个故障注入回归测试。修复后应重新执行 54 项以上测试、真机 `--once` 首次导入/二次去重、无设备和缺少 ADB 两种独立场景、双实例锁测试。
- NG 范围只针对“桌面自动导入与会话监视可作为可靠 MVP 放行”的声明；已成功落盘的 16 个真实会话及其报告仍是有效证据，4 个质量 FAIL 也正确反映旧数据缺陷。本结论不涉及手机采集链路、COLMAP、Gaussian、GPU 或视觉重建质量。

## 桌面端自动导入修复复核

- 2026-09-23 只读复核结论仍为 **NG（范围限定）**，项目代码未修改。第一个阻断项已关闭：直接导入和 watcher 都先在 `.session-<id>.partial` 中拉取、生成质量报告，再通过 `os.replace` 提升为正式目录；报告构建失败故障注入返回 `IMPORT_ERROR`，正式目录不出现，partial 保留，下一次扫描为 `BLOCKED_PARTIAL`。
- 真实 E 盘会话 `session-1789970977358` 的正式目录同时包含 `quality_report.json`、`quality_report.md` 和 `transfer_manifest.json`，partial 不存在；报告逻辑会话名正确，142 个清单文件的字节数和 SHA-256 独立复核无误。报告保持 `FAIL`，原因为旧版 `camera_stream` GSX 缺少包内 `imu/imu.csv`，没有伪装为 PASS；二次扫描为 `ALREADY_IMPORTED` / 总状态 `NO_NEW_SESSION`。
- 历史批量输出独立复核仍为 16 份报告、12 PASS、4 FAIL、传输完整性失败 0、正式目录缺报告 0、partial 0；旧不完整会话继续为 `WAITING_COMPLETE`。
- 第二个阻断项只部分关闭：`run_adb` 已把无法启动 ADB 的 `OSError` 转为 `AdbUnavailableError`，真实缺失可执行文件可正确得到 `ADB_ERROR`；导入阶段本地磁盘 `OSError` 也得到 `IMPORT_ERROR`。但是 `desktop/session_monitor.py` 的文件清单阶段仍显式将原始 `OSError` 映射为 `WAITING_FOR_DEVICE`。独立注入 `list_files -> OSError(5, "local enumeration failure")` 稳定复现总状态和事件状态均为 `WAITING_FOR_DEVICE`。
- 现有 56 项测试全部通过、`compileall` 通过，但新增测试只覆盖导入阶段本地磁盘错误，没有覆盖文件清单阶段的原始本地 `OSError`；因此测试绿灯不能关闭该分类缺口。放行前需将该分支归为 `IMPORT_ERROR` 或明确的本地错误，并增加对应回归测试后重跑。
- NG 范围仅为桌面自动导入/watcher 的可靠性放行；正式终态原子性、真实 FAIL 保留、16 个历史报告和手机采集链路的既有证据不因此失效，也不涉及 COLMAP、Gaussian、GPU 或视觉质量。

## 桌面端自动导入最终验收

- 2026-09-23 由 GPT-5.6-sol 高思考执行只读最终复核，裁决为 **PASS（范围限定）**，项目代码未修改。上一轮唯一阻断项已关闭：`desktop/session_monitor.py` 在 `list_files` 清单枚举阶段捕获原始本地 `OSError` 后记录 `IMPORT_ERROR` 并继续扫描，不再伪装为 `WAITING_FOR_DEVICE`；新增回归测试 `test_local_file_listing_error_is_not_reported_as_device_loss`。
- 原子终态保持成立：导入先在 `.session-<id>.partial` 中完成传输清单、质量报告，再用 `os.replace` 提升为正式目录；报告失败不产生正式目录，保留 partial，二次扫描进入 `BLOCKED_PARTIAL`。`AdbUnavailableError`、设备 `AdbError` 和本地 `OSError` 分别归类为 `ADB_ERROR`、`WAITING_FOR_DEVICE`、`IMPORT_ERROR`。
- 独立重跑 Python unittest 57/57 通过，Python 文件编译检查通过；缺失 ADB 实测为 `ADB_ERROR`，指定未连接设备为 `WAITING_FOR_DEVICE`，第二 watcher 被进程锁拒绝并返回退出码 3。
- 真实原子会话 `session-1789970977358` 的正式目录、JSON/Markdown 质量报告和传输清单均存在，partial 不存在，报告逻辑会话名正确；142 个清单文件逐一字节数和 SHA-256 校验失败 0，二扫为 `ALREADY_IMPORTED` / `NO_NEW_SESSION`。该会话质量为 FAIL 是旧 GSX 缺少包内 IMU 的采集内容问题，watcher 已如实保存终态，不属于导入故障。
- 批量实证保持为 16 份报告、12 PASS/4 FAIL、正式目录缺报告或清单 0、partial 0、报告会话名错配 0；二扫不重复导入，不完整会话继续标记 `WAITING_COMPLETE`。
- 放行范围仅包括桌面端会话发现、ADB 自动导入、传输清单校验、质量报告生成、原子终态、错误分类、重复扫描去重和批量 watcher。非阻断边界：partial 仍需人工检查或清理；混合事件时运维应同时读取 `events`，不能只看顶层状态。本结论不证明 Android 采集质量、COLMAP/Gaussian、GPU、开机常驻、长期无人值守或断电级恢复能力。

## Windows x64 便携包构建（2026-09-23）

- 新增 Windows one-folder/windowed 构建脚本 `scripts/package_windows.ps1` 与使用说明 `docs/WINDOWS_PORTABLE.md`。脚本固定 Python 3.12 x64 / PyInstaller 6.20.0，构建暂存目录在系统临时路径；目标已存在时拒绝覆盖。
- 当前主机 Windows 11 x64 构建成功：目录包 27,899,024 字节；ZIP 11,996,935 字节、998 个条目；旁侧 SHA-256 文件匹配，哈希 `A2B78D0BEF150C8ADECCA422F9A294B141D2AAFDFF1FFAAAE0F8EC5027984571`。
- ZIP 静态检查未发现 `runtime/`、采集会话、GSX/JPEG 样例、测试、samples 或 mobile 目录；随包有 Python、Tcl/Tk、OpenSSL、PyInstaller 与项目许可文本及构建信息。EXE 未签名。
- 冻结版从 Windows `Personal` Known Folder 解析默认数据目录为“文档/GS Capture Studio”；解析失败或目录不可写时要求用户选择，不写回程序包。ADB 保持外部依赖。
- 全量测试 79/79 通过，`compileall` 通过。只放行自动构建和静态包完整性，不代表 EXE 启动、无 Python 环境、只读目录、中文路径、ADB 或真机导入已验证。
- 通过明确参数请求 GPT-5.6-sol/high 的复核工具未能提供可验证的模型身份且曾引用旧包哈希，因此未把该意见登记为指定模型最终验收；当前结论依据本机测试和当前 ZIP 的哈希/目录检查。

## 桌面监视器窗口 MVP 验收（2026-09-23）

- 在 watcher v2 状态契约之上实现 Tkinter 桌面窗口：默认不连接手机，点击“开始监视”后才启动 ADB；提供保存目录、可选序列号、轮询间隔、任务队列、文件计数进度、最近完成文件、质量结果和面向用户的错误建议。
- 队列会从本地正式会话与 `.partial` 重建并与手机发现结果合并；旧快照明确标为历史，不冒充实时在线状态。退出窗口时先发停止请求，等待当前扫描/导入线程结束再销毁。
- GPT-5.6-sol 高思考只读复核为 **PASS（范围限定）**，没有 P1/P2 阻断；自动化全量测试 74/74 通过，`compileall` 通过。UI 状态格式测试 6/6 通过。
- 验收只覆盖代码和自动测试：没有启动真实桌面窗口或连接手机。本轮只读 ADB 检查显示没有已连接设备，因此仍待用户接入后的真实点击、真机监视验证；Windows 打包、GPU、COLMAP 和 Gaussian 继续未验收。
- 待办：桌面窗口复用的 watcher 循环仍会把 JSON 状态写到启动它的控制台；是否为 GUI 增加静默输出选项，需先确认该测试边界后再改。该项不影响状态文件或窗口读取。

## 关联知识

- [[对话知识/2026/2026-09/2026-09-22-GS-Capture-Studio-未匹配帧诊断|未匹配帧根因与安全修复门禁]]
- [[对话知识/2026/2026-09/2026-09-23-GS-Capture-Studio-Windows便携包|Windows x64 便携包构建与静态验收]]
- [[对话知识/00_对话知识索引]]
