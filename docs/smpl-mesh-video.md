# SMPL mesh video (dancer in 3D)

Render the AIST++ motion as a **3D body mesh video** (like the “Fitted SMPL Mesh” in their demo), not just sticks or points.

## What you need

1. **Motion .pkl** – already in the repo if you ran `prepare_test_videos.py --download-aist-motion`  
   → `poc/data/test_fixtures/real/gBR_sBM_cAll_d04_mBR0_ch01.pkl`

2. **SMPL body model** – not in the repo (license). You must:
   - Register at [https://smpl.is.tue.mpg.de/](https://smpl.is.tue.mpg.de/)
   - Download **SMPL for Python Users** → version 1.1.0 (female/male/neutral)
   - Unzip the zip into `poc/backend/data/` so you have `data/SMPL_python_v.1.1.0/smpl/models/` with the `.pkl` files. The script expects symlinks `SMPL_MALE.pkl` and `SMPL_FEMALE.pkl` in that folder (already created if you used the setup from the repo).
   - Or set the path manually: `export SMPL_DIR=/path/to/folder/containing/SMPL_MALE.pkl`

3. **Python deps** (in `poc/backend`). Use a venv if your system Python is managed (PEP 668). For **CPU-only** (smaller download, no GPU):
   ```bash
   cd poc/backend
   python -m venv .venv
   .venv/bin/pip install -r requirements.txt
   .venv/bin/pip install torch --index-url https://download.pytorch.org/whl/cpu
   .venv/bin/pip install smplx trimesh pyrender imageio imageio-ffmpeg
   ```
   Or full GPU torch: `.venv/bin/pip install -r requirements-smpl.txt` (larger download).

## Run

From `poc/backend` (after installing deps and unzipping SMPL into `data/`):

```bash
./scripts/run_smpl_mesh.sh
```

Or manually:

```bash
export SMPL_DIR="$(pwd)/data/SMPL_python_v.1.1.0/smpl/models"
.venv/bin/python scripts/render_smpl_mesh.py
```

Output: `out_3d_demo/smpl_mesh.mp4` (60 fps by default, all frames).

Options:

- `--out PATH` – output video path  
- `--frames N` – render only first N frames (faster test)  
- `--fps 30` – output FPS  
- `--width 640 --height 480` – frame size  
- `--gender male|female|neutral` – SMPL gender  
- `--motion-pkl PATH` – use another motion file  

## Headless / no display

If you have no display (e.g. SSH server), set a software/GPU backend before running:

```bash
export PYOPENGL_PLATFORM=egl
# or, if EGL is not available:
export PYOPENGL_PLATFORM=osmesa
```

Then run the script as above.

## Hardware

Runs on **CPU** (no GPU required). A full 720-frame sequence may take a few minutes. 4 cores and 15 GB RAM are enough. The script normalizes the mesh into a fixed view (camera at 3 units, mesh scaled to fit a 2-unit box) so the dancer is always visible in frame.
