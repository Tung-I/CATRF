<div align="center">

# CATRF: Codec-Adaptive TriPlane Radiance Fields for Volumetric Content Delivery

**CVPR 2026 Findings**

</div>

[![page](https://img.shields.io/badge/Project-Page-blue?logo=github&logoSvg)](https://tung-i.github.io/catrf-cvpr-findings-2026/)
[![arXiv](https://img.shields.io/badge/arXiv-2502.20762-b31b1b.svg)](https://arxiv.org/abs/2605.18054)

<img src="assets/pipeline.png" width="500">

## Overview

CATRF is a standard-codec-in-the-loop training framework for plane-factorized radiance fields. The repository contains two independent implementations:

- **`catrf_static/`** — static scenes. TensoRF backbone, JPEG straight-through estimator (STE). Evaluated on NeRF-Synthetic.
- **`catrf_dynamic/`** — dynamic scenes. TeTriRF-style temporal TriPlane backbone, standard video codec STE (AV1, HEVC, VP9). Evaluated on DyNeRF (N3D).

Training consists of two stages for both branches:

1. **Vanilla pre-training** — train the TriPlane backbone without codec artifacts.
2. **Codec-in-the-loop fine-tuning** — pack feature planes into codec-compatible 2-D canvases, run a standard codec round-trip, unpack and dequantize, then back-propagate through the codec via a straight-through estimator (STE). This adapts the TriPlanes to codec compression artifacts so that rendering quality is preserved after standard-codec compression.

---

## catrf_static

### Prerequisites

- Python 3.8, CUDA 11.8

```bash
conda create -n catrf_static python=3.8
conda activate catrf_static

pip install torch==2.2.1 torchvision==0.17.1 --index-url https://download.pytorch.org/whl/cu118
pip install tqdm scikit-image opencv-python configargparse lpips imageio-ffmpeg kornia
pip install pytorch_msssim compressai wandb tensorboard
```

A full pinned environment is available at `envs/static_requirements.txt`.

### Dataset

This branch supports the **NeRF-Synthetic** dataset (8 scenes: chair, drums, ficus, hotdog, lego, materials, mic, ship).

Download from the [NeRF project page](https://drive.google.com/drive/folders/128yBriW1IG_3NJ5Rp7APSTZsJqdJdfc1) and place under `catrf_static/data/`:

```
catrf_static/
└── data/
    └── nerf_synthetic/
        ├── chair/
        ├── drums/
        ├── ficus/
        ├── hotdog/
        ├── lego/
        ├── materials/
        ├── mic/
        └── ship/
```

### Stage 1 — TensoRF pre-training

Run from the repo root. Replace `nerf_lego` with any of the eight scene names.

```bash
python catrf_static/train/train_tensorf.py \
    --config catrf_static/configs/nerf_lego/pretrain.txt
```

The checkpoint is saved to `catrf_static/logs/tensorf_lego_VM_codec/tensorf_lego_VM_codec.th`.

### Stage 2 — JPEG STE fine-tuning

Available quality levels: `jpeg_q10`, `jpeg_q20`, `jpeg_q35`, `jpeg_q50`, `jpeg_q65`.

```bash
python catrf_static/train/train_ste.py \
    --config catrf_static/configs/nerf_lego/jpeg_q20.txt \
    --ckpt catrf_static/logs/tensorf_lego_VM_codec/tensorf_lego_VM_codec.th \
    --compression --batch_size 65536 \
    --lr_decay_target_ratio 1 --n_iters 30000
```

---

## catrf_dynamic

### Prerequisites

- Python 3.12, CUDA 11.8

```bash
conda create -n catrf_dynamic python=3.12
conda activate catrf_dynamic

pip install torch==2.7.1 torchvision==0.22.1 torchaudio==2.7.1 --index-url https://download.pytorch.org/whl/cu118
pip install -r envs/dynamic_requirements.txt
```

### Dataset

This branch supports the **DyNeRF / N3D** dataset (6 scenes: coffee_martini, cook_spinach, cut_roasted_beef, flame_salmon, flame_steak, sear_steak).

Follow the preprocessing steps from [TeTriRF](https://github.com/wuminye/TeTriRF) and place data under `catrf_dynamic/data/`:

```
catrf_dynamic/
└── data/
    └── n3d/
        ├── flame_steak/
        │   └── llff/
        │       ├── 0/
        │       │   ├── images/
        │       │   └── poses_bounds.npy
        │       ├── 1/
        │       └── ...
        ├── sear_steak/
        └── ...  (one sub-directory per scene, same structure)
```

### Stage 1 — TeTriRF pre-training

`--frame_ids` specifies the GoP. The example below trains a GoP of 15 frames.

```bash
python catrf_dynamic/train/train_vanilla_n3d.py \
    --config catrf_dynamic/configs/n3d/dynerf_flame_steak/video.py \
    --frame_ids 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14
```

### Stage 2 — Video codec STE fine-tuning

Available codec configs per scene:

| Codec | QP options |
|-------|-----------|
| AV1   | qp32, qp38, qp44, qp50 |
| HEVC  | qp28, qp32, qp36, qp40, qp44 |
| VP9   | qp32, qp36, qp40, qp44 |

```bash
python catrf_dynamic/train/finetune_scl_n3d.py \
    --config catrf_dynamic/configs/n3d/dynerf_flame_steak/av1_qp44.py \
    --frame_ids 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14
```

Replace `av1_qp44` with any config listed above, e.g. `hevc_qp32`, `vp9_qp40`.

---

## Acknowledgements

The static branch builds on [NeRFCodec](https://github.com/JasonLSC/NeRFCodec_public) and [TensoRF](https://github.com/apchenstu/TensoRF). The dynamic branch builds on [TeTriRF](https://github.com/wuminye/TeTriRF).

## Citation

If you find this repository useful, please cite CATRF:

```bibtex
@inproceedings{chen2026catrf,
  title={CATRF: Codec-Adaptive TriPlane Radiance Fields for Volumetric Content Delivery},
  author={Chen, Tung-I and Wang, Lingdong and Maji, Subhransu and Sitaraman, Ramesh K},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  pages={457--467},
  year={2026}
}
```
