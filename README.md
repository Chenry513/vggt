# Minimal Multi-View 3D Reconstruction for Robotic Welding

**BCIT — Applied Research Project**  
**In collaboration with Seaspan Shipyards**

---

## Overview

Robotic welding arms today need to be manually programmed for every new part — an engineer physically moves the arm to each weld point and saves its position. This is slow and has to be repeated for every new joint geometry.

This project automates that step. Take 3 photos of a weld joint, run the pipeline, and get real-world weld path coordinates in centimeters — ready to send to a robot arm.

```
3 photos → VGGT 3D reconstruction → point cloud → scale calibration → weld coordinates (cm)
```

The 3D point cloud can be loaded into the **WeldPath Viz viewer** included in this repo to visually inspect the reconstruction and define start and end weld points.

**Live viewer:** https://chenry513.github.io/vggt/

---

## How It Works

**Step 1 — Capture**  
Take 3 photos of the weld joint from slightly different angles. All 3 should show the same face of the joint. The more consistent the camera position and distance, the better the reconstruction.

**Step 2 — 3D Reconstruction**  
[VGGT](https://github.com/facebookresearch/vggt) (by Meta Research) processes the photos and outputs a 3D point cloud — hundreds of thousands of real-world coordinate points representing the surface of the scanned object.

**Step 3 — Scale Calibration**  
VGGT gives shape but not real-world size. The welding fixture table has a grid of holes spaced exactly 5 cm apart. After reconstruction, the script opens a top-down view of the point cloud and you click the centers of two adjacent holes. The script uses that known distance to convert every coordinate to centimeters.

**Step 4 — Export**  
The script saves:
- `reconstruction.ply` — the scaled point cloud, loadable in the viewer or any 3D tool
- `scene_info.json` — scene dimensions and the scale factor applied
- `depth_result_N.png` — a depth map for each input image

---

## Setup

### 1. Clone this repo

```bash
git clone https://github.com/chenry513/vggt.git
cd vggt
```

### 2. Install VGGT

VGGT is not a pip package — it needs to be installed from source inside this folder.

```bash
pip install -e .
```

### 3. Install dependencies

```bash
pip install torch numpy matplotlib scikit-learn scipy opencv-contrib-python Pillow huggingface_hub safetensors einops
```

> On Windows, if you see an OpenMP warning on startup it is harmless. It is suppressed automatically by the script.

---

## Usage

### Configure

Open `weld_pipeline.py` and update the config at the top of `main()`:

```python
image_names  = ["photo1.png", "photo2.png", "photo3.png"]  # your image filenames
CONF_THRESH  = 1.0   # drop low-confidence points (1.0 is a good default)
DOWNSAMPLE   = 1     # set to 4 to reduce file size and speed things up
HOLE_SPACING = 0.05  # fixture hole spacing in meters — 5 cm by default
```

### Run

```bash
python weld_pipeline.py
```

### Scale calibration

After reconstruction, a top-down view of the point cloud opens. Click the centers of two adjacent holes on the fixture table, then close the window. The script calculates the scale factor and applies it to all coordinates.

> Adjacent means directly next to each other horizontally or vertically — not diagonal.

### Outputs

| File | Description |
|------|-------------|
| `reconstruction.ply` | Full point cloud in real-world meters |
| `scene_info.json` | Scene dimensions in cm and scale factor used |
| `depth_result_N.png` | Depth map for each input image |

---

## Viewing the Result

Load `reconstruction.ply` into the viewer at **https://chenry513.github.io/vggt/**

1. Drop the `.ply` file onto the viewer
2. Click **Set Start** and click a point on the joint surface
3. Click **Set End** and click the end of the seam
4. Coordinates are shown in centimeters in the sidebar
5. Press **Play Weld** to animate the path

The viewer runs entirely in the browser — no software installation needed.

---

## Tips for Good Results

- All 3 photos should show the **same face** of the weld joint — do not shoot from completely different sides
- Move the camera **slightly left and right** between shots, keeping roughly the same distance and height
- **Get close** — the joint should fill most of the frame
- **Consistent lighting** helps — avoid harsh shadows across the seam
- A **fixed camera mount** instead of hand-held photos significantly improves accuracy

---

## Current Limitations

**Hand-held photos**  
Camera position and angle vary slightly between shots when taken by hand. This is the main source of coordinate inaccuracy. A fixed motorized camera mount is being built to eliminate this.

**Manual scale calibration**  
The hole-clicking step requires a user each run. This will be automated once the camera mount provides consistent framing.

**Manual weld point selection**  
Start and end points are currently selected by clicking in the viewer. A geometric seam detection algorithm has been built and tested. The next step is training a point cloud model (PointNet++) on labeled weld joint data to make detection fully automatic.

---

## Project Structure

```
vggt/
├── weld_pipeline.py        main reconstruction and calibration script
├── visualize_weld.html     3D viewer (also live at GitHub Pages)
├── requirements.txt        Python dependencies
├── README.md
└── vggt/                   VGGT model code (Meta Research)
```

---

## Acknowledgements

Built at **BCIT** in collaboration with **Seaspan Shipyards**.

VGGT by Meta Research: [github.com/facebookresearch/vggt](https://github.com/facebookresearch/vggt)
