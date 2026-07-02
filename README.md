# Work-Zone RGB-D Safety Dataset 500

This repository contains a 500-image RGB-D work-zone safety dataset built from Record3D/iPhone LiDAR recordings. The final annotations are worker-level: each row describes one worker instance, its bounding boxes, distance/depth label, and semantic safety labels.

## Dataset Summary

- Images: 500 RGB PNG frames, each 960 x 720.
- Final worker instances: 806.
- Source recordings: `Garage1`, `Garage2`, `Garage3`, `Garage4`, `Pave1`, `Pave2`.
- Scene types: `indoor_garage`=498, `outdoor_pavement_closed_road_sim`=308.
- Distance classes: `<3`, `3-6`, `>6` meters.
- Final distance distribution: `<3`=164, `3-6`=312, `>6`=330.
- Valid LiDAR numeric depth: 657 workers.
- Missing LiDAR depth with manual distance band: 149 workers.

## Recommended Files

Use these files for modeling and evaluation:

```text
images/
annotations/worker_gt_compact.csv
annotations/worker_gt_compact.json
annotations/worker_gt_compact_schema.md
annotations/worker_gt_compact_summary.json
```

`worker_gt_compact` is the recommended public/compact table. It removes intermediate audit/debug fields and keeps the properties needed for detection, distance classification/regression, and semantic safety evaluation.

The fuller provenance table is still available as:

```text
annotations/worker_gt_merged.csv
annotations/worker_gt_merged.json
annotations/worker_gt_merged_summary.json
annotations/worker_gt_merged_schema_zh.md
```

Use `worker_gt_merged` only when you need audit fields such as depth source, valid pixel counts, depth failure reasons, semantic verification flags, or notes.

## Compact Annotation Schema

Each row in `annotations/worker_gt_compact.csv` / `.json` is one final worker instance.

| Column | Meaning |
|---|---|
| `worker_key` | Stable worker id, formatted as `{image_id}#{worker_index}`. |
| `image_id` | Image basename without `.png`; the image path is `images/{image_id}.png`. |
| `recording` | Source Record3D recording name. |
| `scene_type` | Coarse scene label: `indoor_garage` or `outdoor_pavement_closed_road_sim`. |
| `worker_index` | 1-based worker index within the image. |
| `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2` | Final full-body worker bbox in 960 x 720 RGB pixel coordinates. |
| `inner_x1`, `inner_y1`, `inner_x2`, `inner_y2` | Smaller depth-sampling bbox inside the full bbox, also in RGB pixel coordinates. |
| `depth_z_m` | Record3D/iPhone LiDAR median depth in meters. Blank/null means no usable LiDAR depth. |
| `manual_distance_band` | Human coarse distance label used when LiDAR depth is unavailable. Values: blank, `<3`, `3-6`, `>6`. |
| `distance_class_3` | Final 3-class distance label for all workers. Values: `<3`, `3-6`, `>6`. |
| `high_visibility_vest` | High-visibility vest label: `true`, `false`, or `uncertain`. |
| `helmet_status` | Helmet state: `worn_secured`, `worn_unsecured`, `worn_unknown`, `absent`, `in_hand`, or `uncertain`. |
| `helmet_worn_binary` | Coarse helmet-worn label: `true`, `false`, or `uncertain`. |
| `orientation` | Worker body orientation: `Facing`, `Side`, `Back`, or `uncertain`. |
| `occlusion_level` | Occlusion level: `none`, `partial`, `heavy`, or `uncertain`. |
| `phone_status` | Phone interaction label: `none`, `call`, `viewing`, or `uncertain`. |

For numeric depth regression/training, use only rows where `depth_z_m` is present. For 3-class distance evaluation, use `distance_class_3` for all workers.

## Distance Labels

The final database uses the updated 3 m / 6 m boundary:

- `<3`: closer than 3 meters.
- `3-6`: 3 meters through 6 meters.
- `>6`: farther than 6 meters.

`distance_class_3` is derived from LiDAR `depth_z_m` when valid. When LiDAR depth is not usable, `manual_distance_band` supplies the final coarse class.

## Semantic Label Counts

- `high_visibility_vest`: `true`=664, `false`=125, `uncertain`=17.
- `helmet_status`: `absent`=198, `uncertain`=17, `worn_unsecured`=79, `worn_secured`=117, `worn_unknown`=382, `in_hand`=13.
- `helmet_worn_binary`: `false`=211, `uncertain`=17, `true`=578.
- `orientation`: `Back`=261, `Side`=246, `Facing`=282, `uncertain`=17.
- `occlusion_level`: `none`=589, `partial`=178, `heavy`=31, `uncertain`=8.
- `phone_status`: `none`=692, `uncertain`=94, `call`=12, `viewing`=8.

## Folder Structure

```text
workzone_rgbd_dataset_500/
  README.md
  images/
    Garage1_000000.png
    ...
  annotations/
    worker_gt_compact.csv
    worker_gt_compact.json
    worker_gt_compact_schema.md
    worker_gt_compact_summary.json
    worker_gt_merged.csv
    worker_gt_merged.json
    worker_gt_merged_summary.json
    worker_gt_merged_schema_zh.md
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

## Provenance Files

`bbox_depth_gt.*` contains bbox/depth annotations before semantic merge. `semantic_gt_final.*` contains semantic labels before the final merge.

`frames.csv`, `detections_yolo_raw.csv`, `dataset_summary.json`, and `final_distribution.md` are early frame-selection / raw YOLO provenance files. They may contain the old preliminary `<3`, `3-5`, `>5` frame-selection bands. Do not use those legacy band fields as final ground truth. The final worker-level distance labels are in `worker_gt_compact` and `worker_gt_merged`.

Raw `.r3d` recordings are not included in this lightweight GitHub package. `annotations/raw_r3d_manifest.csv` records the local source recording metadata.

## Record3D Depth Notes

The original data was captured with the Record3D iOS app. In this project, Record3D `.r3d` archives contain RGB frames plus LiDAR depth/confidence maps:

```text
metadata
sound.m4a
rgbd/{frame_id}.jpg
rgbd/{frame_id}.depth
rgbd/{frame_id}.conf
```

The validated decode used here is:

- `.depth`: LZFSE-compressed little-endian float32 array reshaped to 192 x 256, in meters.
- `.conf`: LZFSE-compressed uint8 array reshaped to 192 x 256.
- Confidence `2`: high-confidence depth point.
- Confidence `1`: usable lower-confidence fallback point.
- Confidence `0`: not used for depth statistics.

For each worker, the final depth value is the median depth inside the inner sampling box, preferring confidence `2` points and falling back to confidence `>=1` when needed.

## Recording Counts

| Recording | Workers |
|---|---:|
| `Garage1` | 76 |
| `Garage2` | 49 |
| `Garage3` | 105 |
| `Garage4` | 268 |
| `Pave1` | 204 |
| `Pave2` | 104 |

