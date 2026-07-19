# Work-Zone RGB-D Safety Dataset

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

## Part 2: Second-Wave RGB + Person Depth Dataset

The fully reviewed second-wave dataset is stored in
[`workzone_rgbd_dataset_wave2/`](workzone_rgbd_dataset_wave2/). It is kept
separate from the original 500-image release above.

### Part 2 Summary

- 1,185 RGB PNG images at 960 x 720 pixels.
- 2,472 reviewed person instances.
- 2,356 persons with a numeric LiDAR depth value.
- 116 persons without usable numeric LiDAR depth; their reviewed coarse distance
  class is retained.
- Distance classes use the final `<3`, `3-6`, and `>6` meter boundaries.
- Class counts: `<3` = 491, `3-6` = 983, and `>6` = 998.

### Recommended Part 2 Files

- `workzone_rgbd_dataset_wave2/images/`: reviewed RGB images.
- `workzone_rgbd_dataset_wave2/annotations.csv`: one row per person, containing
  the final bounding box, numeric depth where available, and distance class.
- `workzone_rgbd_dataset_wave2/dataset_summary.json`: machine-readable dataset
  counts and missing-depth policy.
- `workzone_rgbd_dataset_wave2/README.md`: detailed schema and usage notes.

The compact Part 2 annotation table contains:

```text
worker_key,image_id,image_path,worker_index,
bbox_x1,bbox_y1,bbox_x2,bbox_y2,
depth_z_m,distance_class_3
```

`depth_z_m` is blank when no usable LiDAR point was available for that person.
In those cases, `distance_class_3` still contains the manually reviewed `<3`,
`3-6`, or `>6` range. Part 2 intentionally includes only RGB images and the
final person-level annotations; raw Record3D files, depth maps, YOLO
predictions, review artifacts, and semantic safety labels are not included.
