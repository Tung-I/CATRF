<div align="center">

# CATRF: Codec-Adaptive TriPlane Radiance Fields for Volumetric Content Delivery

**CVPR2026 Findings**

</div>

[![page](https://img.shields.io/badge/Project-Page-blue?logo=github&logoSvg)](https://tung-i.github.io/catrf-cvpr-findings-2026/)
[![arXiv](https://img.shields.io/badge/arXiv-2502.20762-b31b1b.svg)](https://arxiv.org/abs/2605.18054)

<img src="assets/pipeline.png" width="500">

## :book: Overview
CATRF is a standard-codec-in-the-loop training framework for plane-factorized radiance fields. This folder contains both the static-scene and dynamic-scene implementation of CATRF. For static scenes, the static_catrf branch builds on a TensoRF backbone and fine-tunes the learned feature planes through JPEG. For dynamic scenes, this branch builds on a TeTriRF-style temporal TriPlane radiance field backbone and fine-tunes the learned feature planes through real standard video codec roundtrips, including AV1, HEVC, and VP9.

The training of CATRF consists of two major stages:

1. Vanilla TriPlane pre-training: Train the static/dynamic TriPlane radiance field backbone without codec artifacts.

2. Standard-codec-in-the-loop fine-tuning: Quantize and pack learned feature planes into codec-compatible 2D canvases, run a standard codec roundtrip, unpack/dequantize the decoded planes, and optimize the rendered reconstruction quality using a straight-through estimator (STE). This adapts TriPlanes to standard codec compression artifacts. The resulting TriPlanes can then be compressed into compact bitstreams using standard codecs without comprimising rendering quality after decompression.

## :hammer: Usage -- catrf_dynamic

For each step, click it to expand and view details.

<details>
  <summary><font size="5"> Prerequisites</font></summary><br>

* Python 3.12 
* CUDA 11.8 (other versions may also work. Make sure the CUDA version matches with pytorch.)
* Pytorch 
* Environment
    ```
    conda create -n $YOUR_PY_ENV_NAME python=3.12
    conda activate $YOUR_PY_ENV_NAME

    pip install torch==2.7.1 torchvision==0.22.1 torchaudio==2.7.1 --index-url https://download.pytorch.org/whl/cu118
    cd envs
    pip install -r dynamic_requirements.txt
    ```

</details>

<details>

  <summary><font size="5"> Dataset Preprocessing</font></summary><br>

* This code supports the DyNeRF dataset
* Please follow the same dataset preprocessing step of [TeTriRF](https://github.com/wuminye/TeTriRF)
* The expected data folder architecture:
    ```
    catrf_dynamic
    - data
    - - n3d
    - - flame_steak
    - - sear_steak
    - - - llff
    - - - - 0
    - - - - - images
    - - - - - poses_bounds.npy
    - - - - 1
    - - - - 2
    - - - - ...
    ```
</details>

<details>

  <summary><font size="5"> Pre-training</font></summary><br>

* E.g., GoP=15:
    ```
    python catrf_dynamic/train/train_vanilla_n3d.py --config catrf_dynamic/configs/n3d/dynerf_cook_spinach/video.py --frame_ids 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14
    ```
</details>

<details>

  <summary><font size="5"> SCL fine-tuning</font></summary><br>

* E.g., GoP=15:
    ```
    python catrf_dynamic/train/finetune_scl_n3d.py --config catrf_dynamic/configs/n3d/dynerf_flame_steak/av1_qp44.py --frame_ids 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14
    ```
</details>



## Acknowledgements

The static branch builds on [NeRFCodec](https://github.com/JasonLSC/NeRFCodec_public) and a dyndynamic branch builds on [TeTriRF](https://github.com/wuminye/TeTriRF).

## Citation
If you find this repository useful, please cite CATRF:

@inproceedings{chen2026catrf,
  title={CATRF: Codec-Adaptive TriPlane Radiance Fields for Volumetric Content Delivery},
  author={Chen, Tung-I and Wang, Lingdong and Maji, Subhransu and Sitaraman, Ramesh K},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  pages={457--467},
  year={2026}
}