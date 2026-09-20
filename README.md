# DepthWizard — Single-View Height Estimation and 3D Flythrough

**Smart India Hackathon 2026**

| | |
|---|---|
| **Problem Statement ID** | SIH26175 |
| **Organisation** | Indian Space Research Organisation (ISRO) |
| **Theme** | Disaster Management |
| **PS Category** | Software |
| **Team Name** | Idea Spark |

**Live prototype:** https://nikitaborhade90.github.io/depthwizard-sih26175/

---

## The problem

A single optical or satellite image is rich in visual detail but carries no direct height information. Existing monocular depth models predict *relative* depth, not absolute elevation, and are trained on natural images rather than top-down remote-sensing imagery. This domain gap makes it hard to turn everyday satellite or aerial photos into usable elevation data — which matters most exactly when it's least available: during disaster response, when timely terrain understanding drives evacuation, flood-risk, and landslide-risk decisions.

## What DepthWizard does

DepthWizard turns a single optical/aerial image into:

1. A **digital surface model (DSM)** — relative or absolute-metric, depending on whether the input is georeferenced
2. A **textured, explorable 3D terrain** rendered in-browser
3. **Scenario analysis** — preliminary flood inundation and landslide susceptibility, height profiling, and point-elevation queries
4. An **interactive flythrough** for situational awareness

## Try it

Open the [live prototype](https://nikitaborhade90.github.io/depthwizard-sih26175/) — no install needed, runs entirely in the browser.

- Pick a sample scene (**urban riverine** or **hilly terrain**) or upload your own PNG/JPG
- Watch the pipeline run: input → pre-processing → depth estimation → metric calibration → 3D reconstruction → analysis
- Drag the **sun azimuth** slider to see the depth estimate update live
- Use **Probe height** to click any point on the terrain and read elevation, slope, and confidence
- Use **Measure A→B** to draw a height profile between two points
- Raise the **flood water level** slider to see inundated area and exposed footprint update
- Switch camera mode to **Fly (WASD)** or **Flythrough** for an immersive pass over the terrain
- Check the **Validation** panel — the sample scenes carry known ground-truth heights, so the DSM estimate is scored against them (MAE, RMSE, δ<1.25, correlation) using the standard scale-and-shift monocular-depth protocol

## How it works

| Stage | What happens |
|---|---|
| 1. Input | Detects geo-reference (GeoTIFF → absolute path) or its absence (PNG/JPG → relative path); reads ground sample distance where available |
| 2. Pre-processing | Denoising, contrast stretch, tiling/resizing for consistent model input |
| 3. Depth estimation | Fuses two cues: shadow-run length walked back along the sun vector (object height) and shape-from-shading integrated along the same vector (terrain trend). Fusion weight adapts to scene relief |
| 4. Metric calibration | Scale-and-shift transform `H = a·d + b`, anchored to a coarse reference DEM (SRTM/CartoDEM) for base elevation and to scene relief/GCPs for scale |
| 5. 3D reconstruction | Converts the calibrated DSM into a textured mesh, web-rendered with Three.js |
| 6. Analysis & visualisation | Slope, flood inundation, landslide susceptibility, height profiling, confidence masking, and interactive flythrough |

**This browser prototype uses a classical shadow-and-shading estimator** so the full pipeline is inspectable end to end without a backend. In production, stage 3 is served by a fine-tuned Depth Anything V2 model behind a FastAPI endpoint:

```
POST /v1/dsm
  { image, crs?, gcp[]? }
  → rdsm.tif, dsm.tif, conf.tif
```

Every stage downstream of depth estimation — calibration, meshing, flood and slope analysis — is the same code path in both the prototype and the production model, so swapping in the trained model is a single-endpoint change.

## Tech stack (target production system)

- **AI/ML:** Python, PyTorch, Depth Anything V2 fine-tuned on GAMUS
- **Geospatial processing:** GDAL, Rasterio, SRTM/DEM, GCP calibration
- **3D visualisation:** Three.js, WebGL
- **Backend:** FastAPI, model inference API
- **Frontend:** React, responsive UI

## Repository structure

```
├── index.html          # Browser prototype (this is the live site)
├── docs/                # SIH presentation and supporting material
├── backend/             # FastAPI service + Depth Anything V2 inference (in progress)
├── notebooks/           # GAMUS fine-tuning and evaluation notebooks (in progress)
└── README.md
```

## Research and references

1. Yang et al., *Depth Anything: Unleashing the Power of Large-Scale Unlabeled Data*, CVPR 2024 — [arXiv:2401.10891](https://arxiv.org/abs/2401.10891)
2. *Depth Anything V2*, 2024 — [arXiv:2406.09414](https://arxiv.org/abs/2406.09414)
3. GAMUS: Geometry-aware Multi-modal Semantic Segmentation Benchmark for Remote Sensing, 2023 — [arXiv:2305.14914](https://arxiv.org/abs/2305.14914)
4. CartoDEM and Cartosat-1 — [isro.gov.in](https://www.isro.gov.in/Cartosat_1_Completes_a_Decade_in_orbit.html)
5. SRTM: Shuttle Radar Topography Mission, NASA — [earthdata.nasa.gov](https://earthdata.nasa.gov/learn/find-data/near-real-time/srtm)
6. PyTorch documentation — [pytorch.org/docs/stable](https://pytorch.org/docs/stable/)

---

Built for Smart India Hackathon 2026 by **Team Idea Spark**.
