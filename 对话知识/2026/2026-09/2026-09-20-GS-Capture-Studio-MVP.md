---
type: 对话知识
created: 2026-09-20 16:30
updated: 2026-09-20 16:44
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
- 测试结果：8 个 unittest 全部通过；重复运行 GSX、`scene.ply`、`report.json` 哈希一致。

## 待确认或待实测

- 指定版本的 COLMAP、CUDA、PyTorch 和具体 Gaussian trainer 尚未安装或冻结。
- 真实图片集上的相机位姿、稀疏重建、训练、模型质量、GPU 性能和失败诊断均未验收。
- Android Camera2/ARCore/IMU 采集、GSX Builder、PC Studio、Viewer 和安装包属于后续里程碑。

## 可复用知识

- “代码存在”不等于“工具链已支持”；必须将软件闭环、外部工具运行、物理/视觉质量分别记录。
- MVP 验收至少应包含：输入包校验、严格/降级模式、确定性、产物独立校验、失败不覆盖上次成功结果。
- 下一阶段进入真实重建前，先冻结 GSX 1.0 子集、工具版本和受控数据集，再做端到端基线。

## 验收方法

- 先运行 `python -m unittest discover -s tests -v`，当前应为 8 个测试全部通过。
- 再运行 `python scripts/create_demo_gsx.py`、`python gs.py validate samples/demo_room.gsx`，应看到 `valid: true`、`frame_count: 3`。
- 运行 `python gs.py generate samples/demo_room.gsx --backend synthetic --output runtime`，检查 `report.json` 中 `result_kind: synthetic`、`reconstruction_performed: false`、`scene_ply_created: true`，并确认 `scene.ply` 含 `comment reconstruction_performed false`。
- 将同一输入生成到两个目录并比较 `scene.ply`、`report.json` SHA-256，验证确定性。
- 运行 `--require-real --gaussian-command "trainer"`，在未安装 COLMAP 时应返回退出码 4；这证明严格模式不会静默回退。
- 以上通过只代表软件闭环和安全边界通过，不代表真实 COLMAP/Gaussian、GPU 性能或视觉质量通过。

## 关联知识

- [[对话知识/00_对话知识索引]]
