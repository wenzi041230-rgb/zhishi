---
type: 对话知识
created: 2026-09-22 10:14
updated: 2026-09-22 10:41
source: Codex 对话
status: 部分确认
tags:
  - 对话知识
  - GS Capture Studio
  - Android
  - ARCore
  - 性能验收
---

# GS Capture Studio 未匹配帧根因与安全修复门禁

## 用户目标

只做工程决策，不直接修改项目文件；判断 Android Camera2 + ARCore 会话中大量未匹配图像的最可能根因，给出可观测、可归因、可验收的一轮修复方案。

## 已确认结论

- 最近真实会话时长 35.04 秒，`source_image_count=1222`、成功匹配并保存 504、`unmatched_image_count=718`、停止时 `pending_image_count=119`；`504 + 718 = 1222`，因此 pending 不能再与 unmatched 相加。
- 成功保存帧的中位间隔为 66.6 ms，约 15 FPS；成功帧的最大 Pose 时间差为 2.11 ms、p95 约 2.07 ms，均明显小于 ±5 ms 窗口。
- `source_image_count` 在 `acquireLatestImage` 返回后才增加，因此 ImageReader 在此之前被 `acquireLatestImage` 合并/丢弃的帧不会进入 source，也不能解释账面内的 718；它属于另一个尚未观测的上游损失口径。
- 718 是聚合差额，可能同时包含编码队列满后主动丢弃、复制或编码失败、无窗口内 Pose、固定 120 项淘汰和停止时未决项，现有指标不能把它归因到单一故障。

## 决策与依据

- 最可能主因是入口速率明显高于 JPEG/落盘路径的可持续吞吐，并由 `encodeQueueDepth>=4` 触发主动丢弃；source 约 34.9 次/秒，而最终保存约 14.4 帧/秒，现象与约 15 FPS 的下游能力一致。
- `pending=119` 几乎顶到 120，说明固定条数缓存和停止收尾也在放大或掩盖问题。应把 pending 终态化，而不是把 119 留作无法解释的余额。
- ±5 ms 不是当前首要调参对象；但现有 Pose 差值只覆盖成功样本，有幸存者偏差。必须为所有已选帧记录最近 Pose 差值和 TRACKING 状态后，才能正式排除 Pose 缺口。
- 不把“提高队列上限”作为首修。加深队列只吸收短时抖动，不提高持续吞吐，还会增加 NV21 内存和端到端延迟。
- 保持相机和 ARCore 已验证的 30 FPS，不强制相机改 15 FPS。保存路径在入口按传感器时间戳做明确的 15 FPS 选帧，未选帧记为 `policy_skipped`，不再算故障性 unmatched。
- 第一组修复验证保持 JPEG quality 90、队列上限 4，以隔离“显式选帧”效果。若已选帧仍因队列满而丢弃，再单独 A/B 将质量调为 85；只有确认平均编码能力高于 15 FPS、丢弃仅由短突发造成并且内存有余量时，才考虑把队列增至 6 或 8。
- 固定 120 项保留策略后续应改成“覆盖实测 p99 端到端延迟的时间窗 + 硬性内存上限”，具体时间值必须由新增延迟指标决定，不能凭空设定。

## 建议新增的分项计数

- 上游：回调次数、`acquireLatestImage` 成功/空返回、Camera capture-result 数、图像 sensor timestamp；用 capture result 与实际取得图像的时间戳关联估算 `acquireLatestImage` 之前的不可见合并量。
- 准入：`source_acquired`、`policy_selected`、`policy_skipped`、`queue_full_drop`，并记录队列深度最大值及深度驻留分布。
- 处理：YUV 复制成功/失败及 p50/p95/p99 耗时；JPEG 开始、成功、失败、字节数及 p50/p95/p99 编码耗时；从取得图像到编码完成的总延迟。
- 匹配：进入匹配器数量、成功数量、TRACKING/非 TRACKING 数、无 ±5 ms Pose 数、pending 淘汰数、pose sample 淘汰数、停止收尾未决数；所有已选帧都记录最近 Pose 的绝对时间差，而不只记录成功帧。
- 终态：`matched_saved`、`policy_skipped`、`queue_full_drop`、`copy_failed`、`encode_failed`、`no_pose_in_window`、`pending_evicted`、`stop_unresolved`。每个已取得图像必须且只能进入一个终态桶。

## 一轮安全修复顺序

1. 先加入上述可核账指标，保持现有行为跑一条同协议基线，确认 718 的实际组成。
2. 保持 Camera2/ARCore 30 FPS、JPEG 90、队列 4，只在昂贵的 YUV 复制/编码之前按 sensor timestamp 选择 15 FPS；把未选帧明确记作策略跳过。
3. 停止时先关闭新准入，再排空编码任务，用最终 Pose 样本做一次匹配收口，并把每个残留项归入明确失败原因；最终状态不保留无法解释的 pending。
4. 若 `queue_full_drop` 对已选帧仍非零，下一次只改 JPEG 90→85 做 A/B；不要同时改质量和队列。只有证明是短时突发而不是长期过载，才把队列增至 6 或 8，并同时验收内存峰值和延迟。

## 验收门禁

- 计数恒等式逐会话成立，所有 source 都有唯一终态，`unexplained=0`；最终 `pending=0`。
- 在 ARCore `TRACKING` 时，已选帧的 `queue_full_drop=0`、复制失败=0、编码失败=0；已选帧成功保存率至少 99%，输出节奏稳定在目标 15 FPS 附近。
- 对全部已选帧统计最近 Pose 差值；成功帧必须仍在 ±5 ms 内，且无 Pose 的帧要能由 TRACKING 状态或时间差分布解释，不能只看成功样本。
- 先做至少 3 次同场景、同分辨率、同运动方式的 35–60 秒对照，再做一次 5 分钟持续采集，记录队列深度、编码 p99、内存和热状态，防止短测掩盖热降频后的再次过载。
- 产物继续满足 `frames.csv`、JPEG、状态计数和 GSX 帧数一致；图像可解码、尺寸正确，时间戳唯一且递增，Pose 数值有限、四元数有效，传输清单字节数和 SHA-256 全部通过。
- 若只是把 718 改名为 `policy_skipped`，却仍存在已选帧队列丢弃、停止 pending 或计数无法闭合，不算解决；真正的通过条件是“主动选帧可解释，入选帧基本不丢，所有失败可归因”。

## 待确认或待实测

- 当前没有分项计数，尚不能给出 718 中队列满、Pose 缺口、固定缓存淘汰和停止未决各自的精确数量。
- 未提供 JPEG 编码耗时、队列深度时间分布、NV21 占用和设备热状态，暂不能证明 quality 85 或更大队列必然必要。
- 未提供 Camera capture-result 与 ImageReader 图像的一一对应证据，`acquireLatestImage` 上游合并量仍未知。

## 第一轮真机结果与下一步决策

- 真机会话 `session-1790044177576` 约 40 秒：`source=1187`、`policy_selected=581`、`policy_skipped=606`、`encode_submitted=581`、`jpeg_encoded=581`、`encode_queue_drop=0`、`pose_sample=553`、`matched=253`、`selected_unmatched=328`、`no_pose_in_window=328`、`pending_queue_evicted=0`、`pending_at_stop=0`、`pose_queue_evicted=181`；成功匹配最大 Pose 差为 3.99 ms。
- 账本已闭合到关键故障桶：`source = policy_selected + policy_skipped`，`policy_selected = matched + no_pose_in_window`；入选帧编码成功率 100%，本轮不再支持“JPEG 吞吐或编码队列是主要瓶颈”。失败集中在图像与 Pose 时间关联。
- 固定时间戳 15 FPS 的入选速率约 14.5 FPS，而 Pose 样本约 13.8 FPS。若要求一帧使用一个真实且唯一的 Pose，581 个独立入选帧对 553 个 Pose 样本本身就不支持 99% 固定入选帧覆盖；验收分母应改为 Pose 可用候选，不能靠重复 Pose 或凭空补 Pose 达成指标。
- 明确下一步：保持已实测协议不变，再做一次 35–60 秒诊断采集，优先测量所有失败入选帧到最近 Pose 的绝对与有符号时间差、前后包围 Pose 间隔、TRACKING 状态、Pose 时间间隔分布和 Pose 复用情况。最新 `no_pose_delta_min/p95/max` 是必要起点，但应补 p50/p99、分桶和有符号方向，避免三个汇总值掩盖相位峰。
- 若至少 95% 失败帧仅略超窗口（最近 Pose 距离不超过 8 ms，p99 不超过 10 ms，且无约 16–33 ms 的帧周期峰），才允许做 `±5 ms → ±8 ms` 的单变量 A/B，硬上限先不超过 10 ms。
- 若失败距离在 10 ms 以上形成明显分布，尤其聚集在半帧或一帧周期，优先改成 Pose 驱动的候选选择：保持 Camera2/ARCore 已验证的约 30 FPS，由真实 TRACKING Pose 选择最近且未消费的图像候选，仍以严格时间差门槛保存。不要先扩大到可跨相邻相机帧的时间窗。
- 若失败项没有前后 TRACKING Pose、Pose 间隔长或存在跟踪中断，既不能靠扩窗，也不能直接插值；先修 Pose 生产/调度或跟踪连续性。插值只作为更后面的受控实验，必须使用同一连续 TRACKING 段的双侧 Pose、记录派生来源，并通过留一法真值对照和真实重建质量比较后才可放行。
- 采集层验收：逐会话账本 `unexplained=0`、最终 pending 为 0、编码/队列失败为 0、Pose 驱动候选匹配保存率至少 99%、图像与 Pose 一一对应且不复用 Pose、最大时间差不超过所选严格门槛；至少 3 次 35–60 秒和 1 次 5 分钟持续采集均成立。该门禁只证明真实 Pose 采集链路，不等同于 COLMAP/Gaussian 真实重建通过。

## 关联知识

- [[对话知识/2026/2026-09/2026-09-20-GS-Capture-Studio-MVP]]
- [[对话知识/00_对话知识索引]]
