# worker_gt_compact schema

`worker_gt_compact.csv` / `worker_gt_compact.json` is the recommended compact annotation table for this dataset. Each row is one final worker instance.

## Identity and grouping

- `worker_key`: Stable worker id, formatted as `{image_id}#{worker_index}`.
- `image_id`: Image basename without `.png`; the image file is `images/{image_id}.png`.
- `recording`: Source Record3D recording name (`Garage1` ... `Pave2`).
- `scene_type`: Coarse scene label (`indoor_garage` or `outdoor_pavement_closed_road_sim`).
- `worker_index`: 1-based worker index within the image.

## Bounding boxes

All coordinates are in the 960 x 720 RGB image coordinate system, in pixels.

- `bbox_x1`, `bbox_y1`, `bbox_x2`, `bbox_y2`: Final full-body worker bounding box.
- `inner_x1`, `inner_y1`, `inner_x2`, `inner_y2`: Smaller depth-sampling box inside the full bbox.

## Distance and depth

- `depth_z_m`: Record3D/iPhone LiDAR median depth in meters. Blank/null means no usable LiDAR depth for that worker.
- `manual_distance_band`: Human coarse distance label used when LiDAR depth is unavailable. Values are blank, `<3`, `3-6`, or `>6`.
- `distance_class_3`: Final 3-class distance label for all workers. Values are `<3`, `3-6`, or `>6`.

For numeric depth training, use rows where `depth_z_m` is present. For 3-class distance evaluation, use `distance_class_3`.

## Semantic labels

- `high_visibility_vest`: Whether the worker is wearing a high-visibility vest (`true`, `false`, or `uncertain`).
- `helmet_status`: Helmet state (`worn_secured`, `worn_unsecured`, `worn_unknown`, `absent`, `in_hand`, or `uncertain`).
- `helmet_worn_binary`: Coarse helmet-worn label (`true`, `false`, or `uncertain`).
- `orientation`: Worker body orientation (`Facing`, `Side`, `Back`, or `uncertain`).
- `occlusion_level`: Occlusion level (`none`, `partial`, `heavy`, or `uncertain`).
- `phone_status`: Phone interaction label (`none`, `call`, `viewing`, or `uncertain`).
