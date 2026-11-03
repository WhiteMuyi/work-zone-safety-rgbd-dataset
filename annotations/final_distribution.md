# dataset_500 final distribution

> Note: this file records the early frame-selection distribution and may contain the old preliminary `<3`, `3-5`, `>5` bands. It is kept only for provenance. Final worker-level labels now use `<3`, `3-6`, `>6` in `worker_gt_compact` and `worker_gt_merged`.


生成时间：2026-06-25T18:45:48.866527-04:00
最终帧数：500
同 recording 最小时间间隔：0.4 s

## 采用的配额参数

- Garage 且 >5m 箱 top-k：2
- 稀缺箱 top-k（<3m 或 3+人 或 Pave）：12
- 其余箱 top-k：8
- 边界 0.1m 细箱额外 top-k：2

## band target

原始要求约为 `<3=135, 3-5=140, >5=185`；因三者合计不是 500，调参时按比例归一化到最终总数。
归一化目标：`{"<3": 146.7, "3-5": 152.2, ">5": 201.1}`

## loc × band

| loc | <3 | 3-5 | >5 | total |
|---|---|---|---|---|
| Garage | 64 | 84 | 130 | 278 |
| Pave | 75 | 65 | 82 | 222 |

## loc × count

| loc | 1 | 2 | 3+ | total |
|---|---|---|---|---|
| Garage | 115 | 114 | 49 | 278 |
| Pave | 134 | 86 | 2 | 222 |

## recording × band

| recording | <3 | 3-5 | >5 | total |
|---|---|---|---|---|
| Garage1 | 1 | 19 | 31 | 51 |
| Garage2 | 11 | 14 | 24 | 49 |
| Garage3 | 14 | 20 | 31 | 65 |
| Garage4 | 38 | 31 | 44 | 113 |
| Pave1 | 50 | 37 | 38 | 125 |
| Pave2 | 25 | 28 | 44 | 97 |

## recording metadata

| recording | fps | frames | sample_step | sampled_frames |
|---|---:|---:|---:|---:|
| Garage1 | 60.0 | 3292 | 12 | 275 |
| Garage2 | 60.0 | 4051 | 12 | 338 |
| Garage3 | 60.0 | 3651 | 12 | 305 |
| Garage4 | 30.0 | 2256 | 6 | 376 |
| Pave1 | 30.0 | 2686 | 6 | 448 |
| Pave2 | 30.0 | 1814 | 6 | 303 |

## top quota tuning trials

| garage_far_k | scarce_k | default_k | boundary_k | selected_count | <3 | 3-5 | >5 | objective |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 2 | 12 | 8 | 2 | 500 | 139 | 149 | 212 | 0.1173 |
| 2 | 12 | 9 | 2 | 500 | 139 | 149 | 212 | 0.1173 |
| 2 | 11 | 8 | 3 | 500 | 139 | 148 | 213 | 0.1253 |
| 2 | 11 | 9 | 3 | 500 | 139 | 148 | 213 | 0.1253 |
| 2 | 11 | 6 | 4 | 500 | 140 | 145 | 215 | 0.1413 |
| 2 | 10 | 6 | 5 | 500 | 139 | 145 | 216 | 0.1493 |
| 3 | 7 | 7 | 4 | 500 | 127 | 139 | 234 | 0.2933 |
| 3 | 7 | 5 | 5 | 500 | 128 | 136 | 236 | 0.3093 |
