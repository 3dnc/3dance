# Test video for 3D modeling

The test suite uses **synthetic** two-view video (room with moving shapes) so CI does not depend on external assets. For local development or validation with real human motion, you can use the following.

## What you need

- **Two (or more) static-camera videos** of the same scene, same duration, same frame rate.
- **Moving people** in the scene; dance sequences are ideal but any motion is fine.
- **Time-aligned frames** (synchronized start; same number of frames per view).

## Public datasets

| Dataset | Views | Content | License / access |
|--------|-------|---------|-------------------|
| [AIST++](https://google.github.io/aistplusplus_dataset/) | 9 | Dance, 10 genres, 3D keypoints | Research; [downloader](https://github.com/google/aistplusplus_api/blob/main/downloader.py) |
| [TotalCapture](https://cvssp.org/data/totalcapture) | 8 | Freestyle and actions, 60 Hz | Research, registration |
| [MEVA](https://mevadata.org/) | Multiple | Activities, multi-camera | Research |
| [Human3.6M](http://vision.imar.ro/human3.6m/) | 4 | Actions, 3D pose | Academic license |

For **two angles** only: use any multi-view dataset and take two camera streams (e.g. cam0 and cam1). Ensure both clips have the same frame count and are synced; export as two video files (e.g. `view0.mp4`, `view1.mp4`).

## Using real video in the POC

1. Add two file sources in the admin UI (or via API), each pointing to one of the two video files.
2. Start a session; ingestion will write synced frames to `cam0/` and `cam1/` under the session folder.
3. Downstream 3D/pose code can read `frame_00000.png`, `frame_00001.png`, … from each cam directory.

## Preparing test videos

From `poc/backend` (with `PYTHONPATH=.` and optional `POC_DATA_DIR`):

- **Synthetic (no download):**  
  `python scripts/prepare_test_videos.py --synthetic`  
  Writes `test_fixtures/synthetic/view0.mp4` and `view1.mp4` under the POC data dir.

- **Real (AIST++):**  
  `python scripts/prepare_test_videos.py --download-aist`  
  Downloads two views for one AIST++ sequence, then writes synced `test_fixtures/real/view0.mp4` and `view1.mp4`. You must agree to the [AIST Dance Video DB Terms of Use](https://aistdancedb.ongaaccel.jp/terms_of_use/). Non-interactive: `--agree-aist-tos` or `AGREE_AIST_TOS=1`.

- **3D motion (AIST++):**  
  `python scripts/prepare_test_videos.py --download-aist-motion`  
  Downloads the AIST++ motions zip (~306MB), extracts the SMPL 3D motion for the same sequence (`gBR_sBM_cAll_d04_mBR0_ch01`), and writes `test_fixtures/real/gBR_sBM_cAll_d04_mBR0_ch01.pkl`. The pickle contains `smpl_poses` (N, 24, 3), `smpl_scaling`, and `smpl_trans` (N, 3). Use `--motion-timeout` if the download is slow (default 600s).

- **3D keypoints – full body (AIST++):**  
  `python scripts/prepare_test_videos.py --download-aist-keypoints3d`  
  Downloads the AIST++ keypoints3d zip (~834MB), extracts 3D keypoints (17 COCO joints × N frames) for the same sequence, and writes `test_fixtures/real/keypoints3d/gBR_sBM_cAll_d04_mBR0_ch01.pkl`. This is the **dancer transformed to 3D** (all joints over time). Use `--keypoints3d-timeout` if needed (default 900s).

## Viewing a real-world 3D sample (AIST++)

Yes. The sequence `gBR_sBM_c01_d04_mBR0_ch01` (and its 9 camera angles) has downloadable 3D ground truth from AIST++:

| Downloadable file | What it is | How to get it |
|------------------|------------|----------------|
| **Root trajectory** | One 3D point per frame (pelvis path) | `prepare_test_videos.py --download-aist-motion` → `test_fixtures/real/gBR_sBM_cAll_d04_mBR0_ch01.pkl` |
| **Full-body 3D keypoints** | 17 joints × N frames (dancer in 3D) | `prepare_test_videos.py --download-aist-keypoints3d` → `test_fixtures/real/keypoints3d/gBR_sBM_cAll_d04_mBR0_ch01.pkl` |
| **Viewable point cloud** | `points3d.txt` or PLY for the web viewer | `export_aist_3d.py --out out_3d_demo/points3d.txt` (uses motion if keypoints3d not present; uses full-body keypoints if keypoints3d exists). Use `--download-keypoints3d` to fetch keypoints3d then export full body. |

To see the **full-body** dancer in 3D in the point cloud viewer: run `--download-aist-keypoints3d` (once), then `export_aist_3d.py` (it will pick up keypoints3d automatically), then open `out_3d_demo/view_pointcloud.html` (e.g. via `python -m http.server 8765` in `out_3d_demo`).

To render the same motion as a **3D body mesh video** (SMPL), see [smpl-mesh-video.md](smpl-mesh-video.md).

When `test_fixtures/real/view0.mp4` and `view1.mp4` exist, the test `test_prepared_real_two_view_ingestion` runs; otherwise it is skipped. CI uses only synthetic fixtures.

## Synthetic test fixture

The fixture `two_view_room_video` in `poc/backend/tests/conftest.py` (backed by `tests/fixture_videos.py`) generates two MP4s: a simple “room” with moving circles (stand-ins for people) from two simulated camera angles. No download or external data required.
