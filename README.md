# WeldPath Viz — 3D Weld Path Planner

**BCIT — Applied Research Project**
**In collaboration with Seaspan Shipyards**

---

## Overview

WeldPath Viz is a browser-based 3D point cloud viewer and weld path planning tool. It lets you load a reconstructed 3D point cloud of a welding part, visually inspect it, define a weld path by clicking start and end points directly on the part surface, and read out the exact X, Y, Z coordinates.

**Live site:** https://chenry513.github.io/weldpath-viz/visualize_weld.html

---

## Background

This tool was built as part of a research project exploring automated robotic weld path generation using [VGGT (Visual Geometry Grounded Deep Structure From Motion)](https://github.com/facebookresearch/vggt) — a model developed by Meta Research that reconstructs 3D geometry from standard camera images.

The broader pipeline works like this:

```
Photos of weld joint → VGGT 3D reconstruction → Point cloud → WeldPath Viz → Robot coordinates
```

The idea is that instead of manually programming a robotic welding arm for every new joint geometry, you take a few photos of the part, reconstruct it in 3D, and use this visualizer to inspect the result and define where the robot should weld as the project develops.

---

## Features

- **Load any point cloud** — supports `.ply` (ASCII and binary), `.csv`, and `.json` formats
- **Interactive 3D view** — rotate, zoom, and pan around the reconstructed part
- **Click-to-set weld points** — click directly on the part surface to define start and end
- **Weld simulation** — animated torch playback showing the path traversal
- **Coordinate readout** — displays exact X, Y, Z coordinates and direction vector for the robot

---

## Usage

Visit the live site at https://chenry513.github.io/weldpath-viz/visualize_weld.html

### Load your point cloud

Drop one of the following file types onto the viewer:

| Format | Source |
|--------|--------|
| `.ply` | MeshLab, CloudCompare, or the VGGT reconstruction script |
| `.csv` | Any CSV with `x, y, z` columns (plus optional `r, g, b`) |
| `.json` | Output from the `weld_reconstruction.py` script |

### Define the weld path

1. Click **SET START** in the sidebar
2. Click a point on the part surface in the 3D view — a green **S** marker appears
3. Click **SET END** and click the end point — a cyan **E** marker appears
4. Press **PLAY WELD** to simulate the torch moving along the path

### Read the coordinates

The sidebar shows:

```
START
  x: -0.231456
  y:  0.018302
  z:  0.941200

END
  x:  0.198340
  y:  0.021100
  z:  0.938900

DIRECTION VECTOR
  x:  0.998102
  y:  0.012300
  z: -0.001200

LENGTH: 0.429900
```

These coordinates can be passed directly to a robot controller or used as waypoints in a motion planning system.

---

## Controls

| Input | Action |
|-------|--------|
| Left drag | Rotate |
| Scroll | Zoom |
| Right drag | Pan |
| Click a point | Inspect coordinates |
| SET START → click | Set weld start point |
| SET END → click | Set weld end point |
| PLAY WELD | Animate the weld path |
| RESET | Reset camera to default view |

---

## File Format Details

### PLY
Supports both ASCII and binary PLY (little and big endian). Property names `x y z red green blue` or `x y z r g b` are both recognized. Large files are automatically subsampled to 80,000 rendered points for performance.

### CSV
Expects a header row. Recognized column names: `x`, `y`, `z`, `r`, `g`, `b`, `confidence`. All other columns are ignored.

### JSON
Supports two formats:

**Flat point list** (from `weld_reconstruction.py`):
```json
{
  "format": "weld_part_v1",
  "bounding_box": { "x_min": -0.46, "x_max": 0.26, ... },
  "points": [{ "id": 0, "x": 0.12, "y": -0.03, "z": 0.94, "r": 180, "g": 160, "b": 140 }]
}
```

**Weld path** (seam waypoints):
```json
{
  "format": "weld_path_v1",
  "seams": [{ "seam_id": 0, "waypoints": [{ "position": { "x": 0.1, "y": 0.0, "z": 0.9 } }] }]
}
```

---

## Project Structure

```
weldpath-viz/
├── visualize_weld.html      
├── README.md
```

---

## Acknowledgements

Built at **BCIT** in collaboration with **Seaspan Shipyards** as part of a research project into automated robotic weld path generation using VGGT.

VGGT by Meta Research: [github.com/facebookresearch/vggt](https://github.com/facebookresearch/vggt)

---

## License

MIT License
