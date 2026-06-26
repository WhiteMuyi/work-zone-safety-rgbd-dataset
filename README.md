# Work-Zone RGB-D Safety Dataset 500

本文件夹是 Experiment 1 的 GitHub/collaboration 数据包，整理自：

`G:\Research\Proximity\Analysis\Expe1\dataset_500`

数据目标是评估低成本 RGB-D 相机方案在道路作业区安全场景中的可行性：用设备后方视角的 RGB 图像、iPhone LiDAR 深度和人工定稿的 worker 级标注，测试深度/距离估计、PPE 识别和 VLM 问答能力。

## 1. Dataset Summary

- 图像数量：500 张 RGB 图像。
- 图像尺寸：960 x 720，RGB，PNG。
- worker 数量：806 个最终有效 worker。
- 场景：
  - `indoor_garage`：278 张图，498 个 worker。模拟室内/车库类场景。
  - `outdoor_pavement_closed_road_sim`：222 张图，308 个 worker。模拟封闭道路/室外 pavement 场景，正常光照。
- 相机高度：约 1.8 m。
- 相机安装逻辑：模拟安装在自卸卡车后斗/车尾盲区位置的相机。目标是用低成本 RGB-D 模块覆盖车后盲区。
- 图像命名：`{recording}_{frame_id:06d}.png`，例如 `Garage3_000060.png`。
- 推荐主标注文件：`annotations/worker_gt_merged.csv` 或 `annotations/worker_gt_merged.json`。

## 2. Folder Structure

```text
workzone_rgbd_dataset_500/
  README.md
  images/
    Garage1_000000.png
    ...
  annotations/
    worker_gt_merged.csv
    worker_gt_merged.json
    worker_gt_merged_summary.json
    bbox_depth_gt.csv
    bbox_depth_gt.json
    bbox_depth_gt_summary.json
    semantic_gt_final.csv
    semantic_gt_final.json
    semantic_gt_final_summary.json
    frames.csv
    detections_yolo_raw.csv
    raw_r3d_manifest.csv
    dataset_summary.json
    final_distribution.md
```

### 推荐入口

如果只是训练/评估模型，优先使用：

- `images/`
- `annotations/worker_gt_merged.csv`

`worker_gt_merged.csv` 已经按 `worker_key = "{image_id}#{worker_index}"` 把 bbox、depth、distance class 和 semantic GT 合并到每个 worker 一行。

## 3. Data Capture and Record3D / R3D Notes

原始数据由 Record3D iOS App 录制。Record3D 是面向 iPhone/iPad 的 RGB-D / point cloud capture 工具，支持 LiDAR 设备，并提供 RGB-D video、point cloud、USB streaming、PLY/glTF export 等功能。官方页面可参考：

- https://record3d.app/
- https://record3d.app/features.html

本项目使用 Record3D 导出的 `.r3d` 源文件。`.r3d` 在本地表现为 zip-like archive，内部包含：

```text
metadata
sound.m4a
rgbd/{frame_id}.jpg
rgbd/{frame_id}.depth
rgbd/{frame_id}.conf
```

本数据集中 6 个 recording 的 `.r3d` metadata 均显示：

- RGB width / height：`w=960`, `h=720`
- depth width / height：`dw=256`, `dh=192`
- `Garage1`, `Garage2`, `Garage3`：60 FPS
- `Garage4`, `Pave1`, `Pave2`：30 FPS

`.r3d` 原始文件较大，当前 GitHub 数据包没有复制 raw `.r3d` 本体；`annotations/raw_r3d_manifest.csv` 记录了本地 raw `.r3d` 路径、文件大小、FPS、frame count、RGB/depth 分辨率和 archive member 数量。需要重新解码 raw depth/confidence map 时，可按 manifest 回到原始 `.r3d` 文件读取。

### 本项目中已验证的 `.depth` / `.conf` 解码

本项目已有脚本验证 `.r3d` 内部 depth/confidence 的解码方式：

- `.depth`：LZFSE-compressed little-endian float32 array，reshape 为 `(dh, dw)`，即 `(192, 256)`。单位为 meter。
- `.conf`：LZFSE-compressed uint8 array，reshape 为 `(dh, dw)`，即 `(192, 256)`。
- confidence 含义按本项目处理：
  - `2`：高置信深度点。
  - `1`：可用但较低置信深度点。
  - `0`：不用于深度统计。

## 4. Image and Frame Files

### `images/*.png`

500 张从 `.r3d` 中抽出的 RGB frame。每张为 960 x 720 PNG。

### `annotations/frames.csv`

每张图一行，描述被选入数据集的 frame。

主要字段：

- `recording`：原始 recording 名称，`Garage1`、`Garage2`、`Garage3`、`Garage4`、`Pave1`、`Pave2`。
- `loc`：粗场景类别，`Garage` 或 `Pave`。
- `r3d_path`：本地原始 `.r3d` 文件路径。
- `frame_id`：Record3D 内部 frame id。
- `time_sec`：该 frame 在 recording 中的时间戳，秒。
- `person_count`：该 frame 中检测到/最终保留的 worker 数。
- `count_bucket`：人数 bucket，`1`、`2`、`3+`。
- `primary_band`、`primary_dist_conf2`、`primary_dist_conf1`、`primary_gt_distance_m`：早期候选筛选时的主 worker 距离信息。建模时建议使用 `worker_gt_merged.csv` 中的 worker 级字段。
- `quality_score`：早期筛帧质量分数。

### `annotations/detections_yolo_raw.csv`

原始 YOLO person detection 输出，每个 detection 一行。它不是最终 GT，只用于追溯初始检测。

主要字段：

- `recording`, `loc`, `frame_id`, `time_sec`
- `person_rank`：YOLO detection 在该 frame 内的序号。
- `person_count`：该 frame 检测人数。
- `bbox`, `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2`：YOLO bbox，RGB 图像坐标。
- `bbox_area`：bbox 面积。
- `yolo_conf`：YOLO detection confidence。
- `dist_conf2`, `dist_conf1`, `gt_distance_m`, `band`：早期自动深度估计字段。
- `conf2_pixel_count`, `conf1_pixel_count`：bbox sampling region 内不同 confidence 的深度像素数。
- `sharpness`：图像/目标清晰度估计。

最终训练请不要直接使用 `detections_yolo_raw.csv` 作为 GT；应使用 `bbox_depth_gt.csv` 或 `worker_gt_merged.csv`。

## 5. Spatial / Depth Ground Truth

### 深度信息如何生成

最终 bbox 来自 YOLO 初始检测后的人工作业修正。每个 worker 的深度按如下过程生成：

1. 在 RGB 图上使用最终外框 bbox：`bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2`。
2. 为避免腿部、头顶和背景噪声，从外框内缩出一个采样框：
   - x：`0.25` 到 `0.75`
   - y：`0.15` 到 `0.60`
   - 字段为 `inner_x1`, `inner_y1`, `inner_x2`, `inner_y2`
3. 将 RGB 采样框从 960 x 720 映射到 depth map 的 256 x 192：
   - `depth_region_x1`, `depth_region_y1`, `depth_region_x2`, `depth_region_y2`
4. 在对应 depth region 内统计 Record3D LiDAR depth：
   - 优先使用 `confidence == 2` 的有效深度点。
   - 如果没有 `confidence == 2`，退回使用 `confidence >= 1`。
   - 最终 `depth_z_m` 是可用 depth 值的 median。
5. 由 `depth_z_m` 派生三分类距离：
   - `<3`
   - `3-5`
   - `>5`

### 手工距离标签的使用规则

有些 worker 的 LiDAR 点云没有打到人体采样区域，或者没有有效 depth point，因此没有可靠 `depth_z_m`。这部分中有少量距离由人工按三类粗标：

- `<3`
- `3-5`
- `>5`

重要规则：

- 做 depth model training / fine-tuning 时，不要使用人工距离标签。
- 做 test / evaluation 时，可以使用人工三分类距离标签，因为当前只需要验证模型对 `<3`、`3-5`、`>5` 三个区间的判断。
- 推荐用 `worker_gt_merged.csv` 的 `depth_train_usable` 字段筛选训练数据。
- 推荐用 `worker_gt_merged.csv` 的 `distance_eval_usable` 字段筛选距离三分类评估数据。

当前统计：

- LiDAR 数值深度有效：658 worker。
- 高置信 `conf2` 深度：493 worker。
- 低置信 fallback `conf1` 深度：165 worker。
- 无有效 LiDAR 深度：148 worker。
- 人工三分类距离标签：148 worker。
- 可用于 depth training 的 worker：658。
- 可用于三分类距离 evaluation 的 worker：806。
- 无可用距离标签：0。

### `annotations/bbox_depth_gt.csv`

最终 bbox + depth GT，每个 worker 一行。

字段含义：

- `image_id`：图像 ID，对应 `images/{image_id}.png`。
- `recording`：原始 recording。
- `frame_id`：Record3D frame id。
- `time_sec`：该 frame 时间戳。
- `worker_index`：同一张图内 worker 序号。
- `source`：bbox 来源，例如 `yolo`、`edited`、`added`。
- `yolo_conf`：原始 YOLO confidence；人工新增 bbox 可能为空。
- `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2`：最终外框，RGB 图像坐标，单位 pixel。
- `inner_x1`, `inner_y1`, `inner_x2`, `inner_y2`：深度采样框，RGB 图像坐标。
- `depth_region_x1`, `depth_region_y1`, `depth_region_x2`, `depth_region_y2`：采样框映射到 256 x 192 depth map 后的坐标。
- `depth_z_m`：LiDAR median depth，单位 meter；无有效 depth 时为空。
- `distance_band`：由 `depth_z_m` 早期派生的距离文本，可能为 `Very_Close`、`Close`、`Far`。建模建议使用 `worker_gt_merged.csv` 的 `distance_class_3`。
- `depth_status`：`valid_depth` 或 `not_valid_depth`。
- `depth_source`：`conf2`、`conf1` 或 `none`。
- `depth_valid_count`：`confidence >= 1` 且深度有效的像素数。
- `depth_high_conf_count`：`confidence == 2` 且深度有效的像素数。
- `depth_reason`：深度来源/失败原因，例如 `conf_eq_2`、`conf_ge_1`、`no_valid_depth_in_inner_box`。
- `manual_distance_band`：人工粗距离标签，仅用于 evaluation，不用于 depth training。
- `distance_band_gt`：最终距离标签原始字段。注意它混合了旧命名和人工命名；建模建议使用 `distance_class_3`。
- `count_ok`：该 frame 的 worker 数已经人工确认。
- `updated_at`：记录更新时间。

### `annotations/bbox_depth_gt.json`

`bbox_depth_gt.csv` 的 JSON 版本，顶层是 806 个 worker object 的 list。每个 object 的 property 与 `bbox_depth_gt.csv` 同名，含义相同。

### `annotations/bbox_depth_gt_summary.json`

bbox/depth GT 的摘要：

- `image_count`：图像数量。
- `active_gt_rows`：最终有效 worker 数。
- `deleted_rows`：人工 relabel 中删除的无效/重复框数量。
- `count_ok_images`：人数已确认图像数量。
- `depth_status_counts`：`valid_depth` / `not_valid_depth` 计数。
- `manual_distance_band_counts`：人工粗距离标签计数。

## 6. Semantic Ground Truth

语义信息在本数据包中直接作为最终 Ground Truth 使用。推荐读取 `semantic_gt_final.csv/json`，或直接读取合并后的 `worker_gt_merged.csv/json`。

### `annotations/semantic_gt_final.csv`

每个 worker 一行。字段含义：

- `worker_key`：唯一主键，`{image_id}#{worker_index}`。
- `image_id`：图像 ID。
- `worker_index`：同一图内 worker 序号。
- `bbox`：外框坐标，格式为 `x1,y1,x2,y2`。
- `high_visibility_vest`：是否能看到 worker 穿高可视性背心/外套。枚举：`true`、`false`、`uncertain`。
- `helmet_status`：头盔/安全帽状态。枚举：
  - `worn_secured`：安全帽戴在头上，且能看到固定/系好。
  - `worn_unsecured`：安全帽戴在头上，但能看到未系好/松动/吊带。
  - `worn_unknown`：安全帽明确戴在头上，但看不出是否系好。
  - `in_hand`：安全帽在手上，没戴在头上。
  - `absent`：没有戴，也没看到随身拿着。
  - `uncertain`：关键区域看不清，无法判断。
- `orientation`：人体朝向，相对于相机。枚举：`Facing`、`Side`、`Back`、`uncertain`。
- `occlusion_level`：遮挡程度。枚举：`none`、`partial`、`heavy`、`uncertain`。
- `phone_status`：手机状态。枚举：
  - `none`：未看到手机。
  - `call`：手机在耳边通话。
  - `viewing`：低头/手持看手机。
  - `uncertain`：无法判断。
- `verified`：本数据包中均为 `true`。
- `notes`：人工备注。
- `label_source`：标注来源记录，仅用于追溯。建模时可忽略。
- `reviewed_at`：标注/冻结时间。
- `final_gt`：本数据包中均为 `true`。
- `finalized_at`：语义 GT 冻结时间。

### `annotations/semantic_gt_final.json`

JSON 结构：

```json
{
  "metadata": {},
  "workers": []
}
```

`metadata` properties：

- `finalized_at`：语义 GT 冻结时间。
- `source_review_csv`：来源 review CSV。
- `backup_before_freeze`：freeze 前备份。
- `final_csv`、`final_json`：最终输出路径。
- `rows`：worker 行数。
- `unique_worker_keys`：唯一 worker key 数量。
- `verified_true`：verified=true 数量。
- `label_source_counts`：来源计数。
- `semantic_counts`：每个语义字段的类别计数。

`workers` array 中每个 worker object 的 properties 与 `semantic_gt_final.csv` 同名，含义相同。

### 当前语义标签质量判断

比较适合训练/评估：

- `high_visibility_vest`：基础很好。`true=664`、`false=125`、`uncertain=17`。可做 binary classification，训练时建议排除 `uncertain`。
- `helmet_worn_binary`：基础很好。将 `worn_secured`、`worn_unsecured`、`worn_unknown` 合并为 `worn=true`，`absent` 和 `in_hand` 视为 `worn=false`，得到 `true=578`、`false=211`、`uncertain=17`。推荐先做这个二分类。
- `orientation`：数据量也可以，`Facing=282`、`Side=246`、`Back=261`、`uncertain=17`，但定义受视角影响，建议作为第二优先级。

可以评估但不建议第一轮重点训练：

- `occlusion_level`：`none=589`、`partial=178`、`heavy=31`、`uncertain=8`。类别不平衡，heavy 较少。
- 细粒度 helmet strap：`worn_unknown=382` 很多，说明安全帽是否系好在很多图中看不清。若只训练“是否戴在头上”，数据更稳；若训练 secured/unsecured，建议只用 `worn_secured` 和 `worn_unsecured` 中图像足够清楚的样本，并跳过 `worn_unknown`。

暂不建议训练：

- `phone_status`：正样本太少，`call=12`、`viewing=8`，`uncertain=94`。第一轮 pipeline 建议不使用 phone 语义。

## 7. `worker_gt_merged.csv/json`: Recommended Main File

`worker_gt_merged.csv` 是推荐给 collaborator 的主表。它每个 worker 一行，已合并 bbox/depth/semantic GT。

重要字段：

- `worker_key`：唯一主键，`{image_id}#{worker_index}`。
- `image_id`：对应 `images/{image_id}.png`。
- `recording`：原始 recording。
- `scene_type`：`indoor_garage` 或 `outdoor_pavement_closed_road_sim`。
- `frame_id`, `time_sec`, `worker_index`：帧与 worker 序号。
- `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2`：最终 worker bbox。
- `inner_x1`, `inner_y1`, `inner_x2`, `inner_y2`：深度采样 bbox。
- `depth_region_x1`, `depth_region_y1`, `depth_region_x2`, `depth_region_y2`：depth map 坐标。
- `depth_z_m`：LiDAR 数值深度，meter。
- `depth_status`：`valid_depth` / `not_valid_depth`。
- `depth_source`：`conf2` / `conf1` / `none`。
- `manual_distance_band`：人工距离三分类，仅用于 test/eval。
- `distance_class_3`：统一后的三分类距离标签，`<3`、`3-5`、`>5`，空值表示无可用距离标签。
- `distance_label_source`：`lidar_depth_z_m`、`manual_distance_band` 或 `none`。
- `depth_train_usable`：是否可用于训练数值深度/深度三分类。只有 LiDAR 有效且非人工距离标签时为 `true`。
- `distance_eval_usable`：是否可用于三分类距离 evaluation。LiDAR 或人工三分类均可。
- `high_visibility_vest`：安全背心 GT。
- `helmet_status`：细粒度头盔 GT。
- `helmet_worn_binary`：推荐第一轮使用的头盔二分类标签。
- `orientation`：朝向 GT。
- `occlusion_level`：遮挡 GT。
- `phone_status`：手机 GT。
- `semantic_verified`、`semantic_final_gt`：本数据包中均为 true。
- `semantic_label_source`：来源记录，建模时可忽略。
- `notes`：备注。

`worker_gt_merged.json` 的结构为：

```json
{
  "metadata": {},
  "workers": []
}
```

`metadata` properties：

- `dataset_name`：数据集名称。
- `image_count`：图像数量。
- `worker_count`：worker 数量。
- `scene_frame_counts`：场景级图像数量。
- `recording_worker_counts`：每个 recording 的 worker 数。
- `depth_status_counts`：depth 有效性计数。
- `depth_source_counts`：`conf2`、`conf1`、`none` 计数。
- `manual_distance_band_counts`：人工距离标签计数。
- `distance_class_3_counts`：三分类距离分布。
- `distance_label_source_counts`：三分类距离来源计数。
- `depth_train_usable_count`：可用于 depth training 的数量。
- `distance_eval_usable_count`：可用于 distance evaluation 的数量。
- `semantic_counts`：语义标签分布。

`workers` array 中每个 worker object 与 `worker_gt_merged.csv` 字段一致。

## 8. Recommended Modeling Plan

### Step 1: Depth / Distance Model

目标：先跑通 RGB/RGB-D depth pipeline，看低成本 Record3D/iPhone LiDAR 数据能达到什么效果。

建议：

- 用 `worker_gt_merged.csv`。
- depth training 只用 `depth_train_usable == true` 的 658 个 worker。
- 如果做三分类距离，训练仍建议只用 `distance_label_source == lidar_depth_z_m`。
- test/eval 可以用 `distance_eval_usable == true` 的 806 个 worker，其中包含人工三分类标签。
- 不要把 `manual_distance_band` 用进 depth training。
- 如果训练输入是 crop，crop 应来自原始 clean image，不要使用画了 bbox 的 visualization 图。

### Step 2: VLM / Semantic Question Answering

目标：接入 VLM，对 worker crop 或整图提问，测试模型能识别什么、能达到什么程度。

建议第一轮问题：

- 是否穿 high-visibility vest。
- 是否有 helmet worn on head。
- 可选：orientation。

建议暂缓：

- phone_status：正样本太少，先不作为主要指标。
- helmet secured/unsecured：很多样本只能确定戴了帽，但看不出是否系好。先用 `helmet_worn_binary`。

VLM 输入建议：

- 使用原始 RGB 图或 clean worker crop。
- 不要把 bbox 线条画进训练/推理图像像素中；bbox 应作为 metadata 或用于 crop。
- 如果需要定位目标 worker，可给 VLM 原图 + clean crop；不要让框线遮挡 helmet/phone/hand 等细节。

## 9. Practical Notes for GitHub Collaboration

- 本目录已包含 500 张 RGB image 和所有最终 GT CSV/JSON。
- 原始 `.r3d` 文件没有复制进本目录，因为文件很大，不适合普通 GitHub repo。需要 raw depth/confidence map 时，使用 `annotations/raw_r3d_manifest.csv` 回到本地源文件读取；如果必须分享 raw `.r3d`，建议使用 Git LFS 或外部存储。
- `viz_boxes`、relabel app、debug 文件没有放入本数据包。它们不是训练数据。
- 训练语义模型时，建议先用 `high_visibility_vest` 和 `helmet_worn_binary`。
- 训练 depth/distance 模型时，务必遵守 `depth_train_usable`；人工距离标签只用于 test/eval。

## 10. Current Counts

```text
Images: 500
Workers: 806

Scene frames:
  indoor_garage: 278
  outdoor_pavement_closed_road_sim: 222

Depth:
  valid_depth: 658
  not_valid_depth: 148
  conf2: 493
  conf1 fallback: 165
  manual distance labels: 148
  depth_train_usable: 658
  distance_eval_usable: 806

Distance 3-class:
  <3: 164
  3-5: 209
  >5: 433
  no usable distance label: 0

Semantic:
  high_visibility_vest: true 664, false 125, uncertain 17
  helmet_worn_binary: true 578, false 211, uncertain 17
  helmet_status: absent 198, uncertain 17, worn_unsecured 79, worn_secured 117, worn_unknown 382, in_hand 13
  orientation: Facing 282, Side 246, Back 261, uncertain 17
  occlusion_level: none 589, partial 178, heavy 31, uncertain 8
  phone_status: none 692, call 12, viewing 8, uncertain 94
```
