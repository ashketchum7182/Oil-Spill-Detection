# Oil Spill Detection + AIS Vessel Attribution

Detects oil spills in Sentinel-1 SAR satellite imagery and attributes them to
the most likely responsible vessel using nearby AIS (Automatic Identification
System) ship-tracking data.

Trained and evaluated on 14 real Sentinel-1 SAR image/mask pairs from Gulf of
Mexico spills (2018-2020), from the "Oil Spill Segmentation" dataset by
William Alberto Ramirez (Zenodo record 4672426, CC-BY-4.0). See
[Dataset](#dataset).

## What it does

Given a SAR image patch, the pipeline decides whether it shows an oil spill,
and if so, converts the detection to a real-world location and scores nearby
vessels by how likely each one is to be the source.

## Dataset

**Oil Spill Segmentation** — William Alberto Ramirez, Zenodo, 2021.
DOI: [10.5281/zenodo.4672426](https://doi.org/10.5281/zenodo.4672426) ·
Record: https://zenodo.org/records/4672426 · License: CC-BY-4.0

Sentinel-1A GRD (VV polarization) SAR images paired with oil-spill masks for
23 Gulf of Mexico spill scenes (2018-2020). The masks were drawn from
high-confidence NOAA spill reports; the radar images come from the Copernicus
Open Access Hub.

**Why this dataset:**

- **Real SAR imagery with verified labels.** Oil dampens surface ripples and
  shows up as dark patches in SAR, which works day or night and through
  cloud. The masks come from confirmed NOAA reports rather than guesses, so
  the detector learns from real spills.
- **Georeferenced images.** Each source image carries UTM (EPSG:32616)
  metadata, which is what lets stage 3 turn a detected patch into real
  lat/long for AIS matching.
- **Gulf of Mexico.** A busy shipping and offshore-drilling region, which
  fits the goal of attributing spills to nearby vessels.
- **Ready-made patch index.** The dataset ships the patch-coordinate CSVs
  (`dataframe_*_256_90.csv`) that stage 1 uses to crop 256x256 patches.
- **Open license.** CC-BY-4.0 allows reuse with attribution.

**Getting it:** the archive (`Radar_data.rar`, ~490 MB) is too large for
GitHub and is not stored in this repo. `OilSpill_backend.ipynb` downloads and
extracts it automatically on first run. To run the pipeline locally,
download it from the Zenodo record above and extract it so these paths exist:

```
train/images/*.tif
train/dataframe_train_dataset_256_90.csv
train/dataframe_val_dataset_256_90.csv
```

## The 5-stage pipeline

All stages live in `oil_spill_pipeline.py`.

1. **Dataset loading and patch extraction** — reads the dataset's own
   patch-coordinate CSVs and crops 256x256 patches from the source SAR
   images, using a verified (row, col)-swapped, center-anchored coordinate
   convention.
2. **Feature extraction and detection** — computes 9 texture features per
   patch (mean, std, min, max, skew, kurtosis, edge density, GLCM contrast,
   GLCM homogeneity) and classifies oil vs. no-oil with a class-balanced
   random forest (`oil_spill_detector.pkl`).
3. **Geolocation** — converts a detected patch's pixel coordinates to real
   lat/long via each source image's UTM transform (all 14 source images
   carry UTM Zone 16N / EPSG:32616 metadata).
4. **AIS correlation and suspicion scoring** — scores nearby vessels using a
   hand-weighted formula (distance, bearing toward the spill, AIS reporting
   gap spanning the spill time) rather than a trained classifier, since no
   labeled "confirmed guilty vessel" dataset exists to train one on.
5. **Multi-tier escalation** — classifies the top suspicion score into
   `AUTO-ESCALATE` (>0.8), `HUMAN REVIEW` (>0.4), or `LOG ONLY`.

See the module docstring and inline `VERIFIED`/`REJECTED` comments in
`oil_spill_pipeline.py` for the empirical reasoning behind each design
decision.

## Project layout

- `oil_spill_pipeline.py` — the 5-stage pipeline described above; runnable
  standalone (see below).
- `oil_spill_detector.pkl` — the pretrained random forest classifier
  (scikit-learn 1.6.1) used by stage 2.
- `OilSpill_backend.ipynb` — wraps the pipeline in a FastAPI backend and
  serves it from Google Colab through an ngrok tunnel (installs `fastapi`,
  `pyngrok`, `uvicorn` and runs `uvicorn`/`ngrok` inside notebook cells).
  It is not currently packaged as a standalone script.
- `oil_spill_live_app.html` — a static, dependency-free HTML/JS frontend.
  Open it directly in a browser, paste in the ngrok URL printed by the
  notebook's backend cell, and it will call that backend's `/health`,
  `/images`, `/analyze`, and related endpoints.
- Large source data (`Radar_data.rar`, the extracted `train/` folder of
  images and CSVs) is fetched at runtime and is intentionally not tracked in
  this repo — see `.gitignore`.

## Running locally

The pipeline itself (stages 1-5) is a plain Python module and can be run or
imported directly:

```bash
pip install -r requirements.txt
python oil_spill_pipeline.py
```

The `if __name__ == "__main__":` block in `oil_spill_pipeline.py` expects the
dataset's `train/dataframe_train_dataset_256_90.csv`,
`train/dataframe_val_dataset_256_90.csv`, and the `train/images/` directory
to be present locally (these are the large, runtime-fetched files excluded
from git — see `.gitignore`). It trains a fresh detector and runs one example
detection end-to-end.

To use the pretrained model instead of retraining:

```python
import joblib
from oil_spill_pipeline import run_pipeline

detector = joblib.load("oil_spill_detector.pkl")
result = run_pipeline(detector, image_path, x, y, fleet)
```

The full interactive demo (FastAPI backend + `oil_spill_live_app.html`
frontend) currently only runs by opening `OilSpill_backend.ipynb` in Google
Colab and running its cells, which starts the backend and exposes it via an
ngrok tunnel; there is no local-only entry point for the backend yet.

## Docker

Not yet done. Docker packaging is planned as a separate, later task.

## Roadmap & business potential

See [SCALING_AND_BUSINESS.md](SCALING_AND_BUSINESS.md) for the scaling
roadmap and target industries/customers (India and international).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
