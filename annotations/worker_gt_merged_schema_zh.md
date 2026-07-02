# worker_gt_merged schema

Final database entry point:

- `annotations/worker_gt_merged.csv`
- `annotations/worker_gt_merged.json`

Granularity: one row/object per worker instance in one image.

Distance classes use 3 m and 6 m boundaries: `<3`, `3-6`, `>6`.

## Identity / Frame Keys

- `worker_key`: Unique worker id, formatted as `{image_id}#{worker_index}`.
- `image_id`: Image stem. The image path is `images/{image_id}.png`.
- `recording`: Source recording name, e.g. `Garage1`, `Pave2`.
- `scene_type`: Coarse scene group, `indoor_garage` or `outdoor_pavement_closed_road_sim`.
- `frame_id`: Record3D frame id inside the source `.r3d` recording.
- `time_sec`: Timestamp in the source recording, in seconds.
- `worker_index`: Worker index within the image.

## BBox / Sampling Geometry Keys

- `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2`: Final worker bounding box in RGB image pixels.
- `inner_x1`, `inner_y1`, `inner_x2`, `inner_y2`: Inner RGB sampling box used for LiDAR depth statistics.
- `depth_region_x1`, `depth_region_y1`, `depth_region_x2`, `depth_region_y2`: Inner sampling box mapped to the 256 x 192 depth map.

## Depth / Distance Keys

- `depth_z_m`: Numeric LiDAR median depth in meters. Empty if no valid depth was available.
- `depth_status`: `valid_depth` or `not_valid_depth`.
- `depth_source`: `conf2`, `conf1`, or `none`.
- `depth_valid_count`: Count of valid `confidence >= 1` depth pixels in the sampling region.
- `depth_high_conf_count`: Count of high-confidence `confidence == 2` depth pixels in the sampling region.
- `depth_reason`: How depth was obtained or why it failed, e.g. `conf_eq_2`, `fallback_conf_ge_1`, `no_valid_depth_in_inner_box`.
- `manual_distance_band`: Human distance label used when LiDAR depth was unavailable or manually corrected. Values are empty, `<3`, `3-6`, `>6`.
- `distance_band_gt_original`: Source distance label before conversion to the unified class. For numeric LiDAR rows this may be `Very_Close`, `Close`, or `Far`; for manual rows it is `<3`, `3-6`, or `>6`.
- `distance_class_3`: Recommended final distance class for modeling/evaluation. Values: `<3`, `3-6`, `>6`.
- `distance_label_source`: `lidar_depth_z_m`, `manual_distance_band`, or `none`.
- `depth_train_usable`: `true` only when numeric LiDAR depth is available. Use this for depth regression/training.
- `distance_eval_usable`: `true` when final 3-class distance GT is available. Currently true for all 806 workers.

## Semantic / PPE Keys

- `high_visibility_vest`: `true`, `false`, or `uncertain`.
- `helmet_status`: Fine helmet label: `worn_secured`, `worn_unsecured`, `worn_unknown`, `in_hand`, `absent`, or `uncertain`.
- `helmet_worn_binary`: Recommended binary helmet target derived from `helmet_status`: `true`, `false`, or `uncertain`.
- `orientation`: Worker orientation relative to the camera: `Facing`, `Side`, `Back`, or `uncertain`.
- `occlusion_level`: `none`, `partial`, `heavy`, or `uncertain`.
- `phone_status`: `none`, `call`, `viewing`, or `uncertain`.
- `semantic_verified`: Whether the semantic row was verified/frozen.
- `semantic_final_gt`: Whether the semantic row is part of final GT.
- `semantic_label_source`: Label provenance, e.g. `human_verified` or `gpt_prelabel`.
- `notes`: Human notes.

## Recommended Modeling Columns

For most modeling/evaluation work, start with:

- Identity/split: `worker_key`, `image_id`, `recording`, `scene_type`, `worker_index`
- Localization: `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2`
- Distance: `depth_z_m`, `depth_status`, `manual_distance_band`, `distance_class_3`, `distance_label_source`, `depth_train_usable`, `distance_eval_usable`
- Semantic targets: `high_visibility_vest`, `helmet_status`, `helmet_worn_binary`, `orientation`, `occlusion_level`, `phone_status`

## Low-Value / Mostly Audit Columns

These are useful for reproducibility and debugging, but usually not needed as model targets:

- `inner_x1`, `inner_y1`, `inner_x2`, `inner_y2`
- `depth_region_x1`, `depth_region_y1`, `depth_region_x2`, `depth_region_y2`
- `depth_valid_count`, `depth_high_conf_count`, `depth_reason`
- `distance_band_gt_original`
- `semantic_verified`, `semantic_final_gt`, `semantic_label_source`, `notes`

Recommendation: keep the full `worker_gt_merged` as the canonical database, and create a lighter training view only when needed. Do not delete audit columns from the canonical version yet.
