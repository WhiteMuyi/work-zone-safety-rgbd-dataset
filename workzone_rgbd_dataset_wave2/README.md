# Work-Zone RGB + Person Depth Dataset — Wave 2

This folder is the finalized second-wave data collected for the work-zone person
detection and distance project. It contains only reviewed RGB images and a compact
person-level annotation table.

## Contents

```text
workzone_rgbd_dataset_wave2/
  README.md
  dataset_summary.json
  annotations.csv
  images/
    2person-garage_000009.png
    ...
```

## Dataset Summary

- Reviewed source frames: 1,187 / 1,187.
- Kept RGB images: 1,185.
- Excluded frames: 2.
- Final person instances: 2,472.
- Persons with numeric LiDAR depth: 2,356.
- Persons without numeric LiDAR depth: 116.
- Image format and size: RGB PNG, 960 × 720.
- Distance classes: `<3`, `3-6`, `>6` meters.

## Annotation Table

`annotations.csv` contains one row per person.

| Column | Meaning |
|---|---|
| `worker_key` | Stable person identifier, formatted as `{image_id}#{worker_index}`. |
| `image_id` | Image basename without `.png`. |
| `image_path` | Relative path to the RGB image. |
| `worker_index` | 1-based person index within the image. |
| `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2` | Final reviewed person bbox in 960 × 720 RGB pixel coordinates. |
| `depth_z_m` | Person-level LiDAR depth in meters. Blank means no usable numeric LiDAR depth. |
| `distance_class_3` | Final reviewed distance range: `<3`, `3-6`, or `>6`. |

The distance boundaries are:

- `<3`: depth below 3 meters.
- `3-6`: depth from 3 through 6 meters, inclusive.
- `>6`: depth above 6 meters.

When numeric LiDAR depth is available, `distance_class_3` is derived from
`depth_z_m`. When numeric LiDAR depth is unavailable, `depth_z_m` is blank and
`distance_class_3` retains the manually reviewed coarse distance range.

## Scope

This compact Wave 2 package intentionally excludes depth maps, Record3D archives,
raw YOLO predictions, depth-sampling windows, review queues, and semantic safety
labels. Only final RGB images, person bboxes, numeric depth where available, and
the three-class distance label are included.
