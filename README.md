# Cyclops

This is the code repository for the CoRL 2026 paper **“Cyclops: LiDAR as a Camera That Dreams in Color.”**

[![Cyclops](cover_vedio.png)](https://youtu.be/QiCe5U_9kOg)

<p align="center">
  <a href="https://arxiv.org/pdf/2608.16264">
    <img src="https://img.shields.io/badge/arXiv-2608.16264-B31B1B?style=for-the-badge&amp;logo=arxiv&amp;logoColor=white" alt="Paper">
  </a>
  <a href="https://youtu.be/QiCe5U_9kOg">
    <img src="https://img.shields.io/badge/YouTube-Project%20Video-FF0000?style=for-the-badge&amp;logo=youtube&amp;logoColor=white" alt="Project video">
  </a>
  <a href="./cyclops">
    <img src="https://img.shields.io/badge/Code-Cyclops-3776AB?style=for-the-badge&amp;logo=python&amp;logoColor=white" alt="Code">
  </a>
  <a href="./cyclops_task">
    <img src="https://img.shields.io/badge/Demos-Downstream%20Tasks-2EA44F?style=for-the-badge" alt="Downstream demos">
  </a>
</p>

## Overview
The complete pipeline contains two processing stages:

1. **Stage I — intensity densification:** sparse projected LiDAR intensity is converted into a dense intensity image. For this stage, please refer to [Super-LiDAR-Intensity](https://github.com/IMRL/Super-LiDAR-Intensity).
2. **Stage II — temporal colorization:** dense intensity images are transported to the RGB domain with few-step latent bridge matching and previous-frame context.

The current [`cyclops/`](./cyclops) package implements Stage II and expects Stage-I outputs under an `intensity_dense/` directory.

## Prerequisites
### Create the Conda environment

From the repository root:

```bash
conda env create -f cyclops.yaml -y
conda activate cyclops
```
## Training data

Training uses paired camera images and dense intensity maps with matching numeric frame indices:

```text
data/
├── train/
│   ├── sequence_001/
│   │   ├── camera/
│   │   │   ├── camera_image_0.png
│   │   │   ├── camera_image_1.png
│   │   │   └── ...
│   │   └── intensity_dense/
│   │       ├── intensity_map_0.png
│   │       ├── intensity_map_1.png
│   │       └── ...
│   └── sequence_002/
│       ├── camera/
│       └── intensity_dense/
└── val/
    └── sequence_101/
        ├── camera/
        └── intensity_dense/
```
## Training

```bash
cd cyclops
bash examples/training/run_train_two_phase.sh
```

or:

```bash
python examples/training/train.py examples/training/config/intensity_phase1.yaml
python examples/training/train.py examples/training/config/intensity_phase2.yaml
```
## Inference


```text
data/test/
└── sequence_001/
    └── intensity_dense/
        ├── intensity_map_0.png
        ├── intensity_map_1.png
        └── ...
```

Run from `cyclops/`:

```bash
python examples/inference/infer.py \
  --ckpt_path last.ckpt \
  --config_yaml_path config.yaml \
  --data_root ./data/test \
  --out_root ./outputs/generated \
  --num_steps 4
```

### Cyclops for Specific Applications

Run one task at a time:

```
python demo.py --task semantic_segmentation
python demo.py --task lane_detection
python demo.py --task visual_navigation
python demo.py --task point_cloud_colorization
```
**Semantic Segmentation:**
The semantic segmentation demo is built on [SAM 2](https://github.com/facebookresearch/sam2) and uses the included SAM 2.1 Hiera-S checkpoint. It propagates LabelMe point prompts through an ordered RGB sequence.

For a custom sequence:

```bash
python demo.py \
  --image_dir /path/to/rgb_sequence \
  --labelme_dir /path/to/labelme_prompts \
  --out_dir outputs/custom
```

**Lane Detection:**
The lane-detection demo is built on [LaneATT](https://github.com/lucastabelini/LaneATT).  Before the first run, compile the bundled native CUDA NMS extension once:

```bash
cd cyclops_task
python -m pip install --no-build-isolation ./lane_detection/lib/nms

cd lane_detection
python demo.py
```
The model weights are available at [here](https://drive.google.com/drive/folders/1Q6RIBZEKzB_hO-7trCrHkn22zB2jFd8m?usp=drive_link).

**Visual Navigation:**
The visual-navigation demo is built on [ViNT](https://github.com/robodhruv/visualnav-transformer) 

```bash
cd cyclops_task/visual_navigation
python demo.py
```

The model weights are available at [here](https://drive.google.com/drive/folders/1Q6RIBZEKzB_hO-7trCrHkn22zB2jFd8m?usp=drive_link).

**Point Cloud Colorization:**
The point-cloud release provides two ROS 1 C++ utilities:

- [`extract_odom.cpp`](./cyclops_task/point_cloud_colorization/cyclops_ros/src/extract_odom.cpp): extracts odometry aligned with accumulated LiDAR frames from rosbag files.
- [`colorize_pcd_to_global_bag.cpp`](./cyclops_task/point_cloud_colorization/cyclops_ros/src/colorize_pcd_to_global_bag.cpp): assigns Cyclops RGB values to the corresponding point clouds and writes camera images and colored point clouds into a rosbag.

Required ROS-side dependencies include ROS 1, rosbag, `livox_ros_driver2`, PCL, OpenCV, Boost, and Eigen.

The provided `cyclops_ros` package can be placed in a catkin workspace and built with the declared ROS dependencies:

```bash
mkdir -p ~/cyclops_ws/src
cp -r cyclops_task/point_cloud_colorization/cyclops_ros ~/cyclops_ws/src/
cd ~/cyclops_ws
catkin build
source devel/setup.bash
```

```bash
rosrun cyclops_ros extract_odom \
  /path/to/input_rosbags \
  /path/to/processed_output

rosrun cyclops_ros colorize_pcd_to_global_bag \
  /path/to/processed_output \
  /path/to/output_bags
```
We provide a demo [rosbag](https://drive.google.com/drive/folders/1Q6RIBZEKzB_hO-7trCrHkn22zB2jFd8m?usp=drive_link) for cyclops Point Cloud Colorization; you can visualize it with RVIZ

```
rosbag play --pause camera_and_colored_pointcloud.bag
```


## Acknowledgements
Part of the code references the implementation of the [Latent Bridge Matching](https://github.com/gojasper/LBM). The downstream release uses components from [SAM 2](https://github.com/facebookresearch/sam2), [LaneATT](https://github.com/lucastabelini/LaneATT), and [ViNT](https://github.com/robodhruv/visualnav-transformer). We thank the authors for their awesome work!

## Citation

If you find Cyclops useful in your research, please cite:

```bibtex
@inproceedings{gao2026cyclops,
  title     = {Cyclops: LiDAR as a Camera That Dreams in Color},
  author    = {Gao, Wei and Shu, Jian and Zhao, Mingle and Ghaffari, Maani and Kong, David and Xu, Chengzhong and Kong, Hui},
  booktitle = {Proceedings of the 10th Conference on Robot Learning},
  year      = {2026}
}
```


