---
layout: post
title: "Pedicel-Targeted Strawberry Harvesting via Behavioral Cloning on a Low-Cost Teleoperation Platform"
date: 2026-08-28 09:00:00-0400
description: "A complete guide to training a custom Action Chunking with Transformers (ACT) policy for imitation learning in robotics."
tags: ros2 robotics imitation-learning agriculture computer-vision edge-computing -target-harvesting
categories: robotics
thumbnail: assets/img/strawberry_pedicel.jpg
---

August 2026

`imitation-learning` `lerobot` `ACT` `behavioral-cloning` `SO-ARM101` `jetson` `agriculture` `edge-deployment`

---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <div class="embed-responsive embed-responsive-16by9">
            <iframe 
                class="embed-responsive-item rounded z-depth-1"
                src="https://youtu.be/12cEq4G04js"
                allowfullscreen>
            </iframe>
        </div>
    </div>
</div>
<div class="caption"> 
</div>

---

## 🍓 🤖 Background

Strawberry harvesting remains one of the most labor-intensive operations in precision agriculture, with producers spending over $1 billion annually on selective harvesting alone. The pedicel is a slender stem connecting the fruit to the plant and measures only 1.3–3 mm in diameter and is visually similar to the surrounding calyx and foliage, making it one of the most challenging small-target manipulation tasks in agricultural robotics.

Classical approaches to this problem rely on training a vision model for detection, then separately integrating that with motion planning and contact control, three independently engineered components that accumulate error at each handoff. This work takes a fundamentally different approach: instead of programming what to do, we show the robot how.

Using a HiWonder SO-ARM101 bilateral teleoperation platform, we collected human demonstrations of the full reach–grasp–pull–retract sequence and trained an Action Chunking with Transformers (ACT) policy directly from synchronized wrist-camera observations and joint-space actions. The trained policy was deployed autonomously on a Jetson Orin AGX at 15–20 Hz, no cloud compute, no intermediate perception stage, no hand-coded motion planning.

> *"Not farm-ready, yet. From hard-coded to human-taught."*

This write-up documents the full pipeline: hardware setup, bilateral teleoperation, dataset collection, training on HiPerGator, FP16 model conversion, and edge deployment on Jetson Orin. If you are working with the SO-ARM101 and want to implement your own imitation learning pipeline, you can clone the repository and follow the steps below.

---

## 🍓 Overview

This tutorial covers the full behavioral cloning pipeline for pedicel-targeted strawberry harvesting:

- Setting up the **SO-ARM101** leader–follower bilateral teleoperation system
- Collecting demonstration data using **LeRobot**
- Training an **ACT policy** with ResNet-50 visual backbone on HiPerGator B200 GPU
- Converting to **FP16** and deploying on **Jetson Orin AGX** at 15–20 Hz
- Evaluating autonomous inference and understanding failure modes

> **✅ PRO TIP:** The most time-consuming parts are calibration and dataset collection. Budget requires a few hours for setup and additional hours could be spent on installing relevant packages on the Nvidia Jetson Orin. For the new users, performing teleoperation could take a hours of practice as it requires one to carefully guide the arm while the policy picks up the visuomotor policy for an efficient motion-planning. For quality data collection, it could take somewhere around 5-6 hours to collect 150+ quality demonstrations. Rushing the demonstrations produces a poor policy, consistency matters more than speed.

**Dataset:** [🤗 strawberry_pedicel_grasp_teleoperation](https://huggingface.co/datasets/nitindominicrai/strawberry_pedicel_grasp_teleoperation) — 131 episodes, 227K frames, publicly available.

**Pre-trained model:** [🤗 act_strawberry_pedicel](https://huggingface.co/nitindominicrai/act_strawberry_pedicel) — download and deploy directly on your SO-ARM101.

---

## 📋 Table of Contents

1. [Hardware and Software Requirements](#1-hardware-and-software-requirements)
2. [Installation](#2-installation)
3. [Arm Calibration](#3-arm-calibration)
4. [Teleoperation — Verifying Setup](#4-teleoperation--verifying-setup)
5. [Dataset Collection](#5-dataset-collection)
6. [Training on HiPerGator](#6-training-on-hipergator)
7. [Converting to FP16 for Edge Deployment](#7-converting-to-fp16-for-edge-deployment)
8. [Autonomous Inference on Jetson Orin](#8-autonomous-inference-on-jetson-orin)
9. [Benchmarking Inference Speed](#9-benchmarking-inference-speed)
10. [Failure Analysis and Known Limitations](#10-failure-analysis-and-known-limitations)

---

## 1. Hardware and Software Requirements

| Component | Details |
|---|---|
| 🤖 **Robot** | HiWonder SO-ARM101 (leader + follower arms) |
| 🎮 **Servos** | Feetech STS3215 (×6 per arm) |
| 📷 **Camera** | USB wrist-mounted camera (640×480, 30 fps) |
| 💻 **Edge compute** | NVIDIA Jetson Orin AGX (64 GB) |
| 🐧 **OS** | Ubuntu 20.04, JetPack 5.x, CUDA 11.4 |
| 🧠 **Training GPU** | NVIDIA B200 (HiPerGator HPC) |
| 🐍 **Language** | Python 3.12 |
| 📦 **Framework** | LeRobot (HuggingFace) |
| 🏋️ **Policy** | ACT — Action Chunking with Transformers |

> **⚠️ Important:** PyTorch for Jetson Orin must be built from source for CUDA 11.4 and aarch64. The standard pip install will not work on JetPack 5.x. Allow 2–4 hours for the first-time build if building from source.

---

## 2. Installation

### Step 1: Clone Repository

```bash
git clone https://github.com/nitin-dominic/autonomous_strawberry_pedicel_harvesting
cd autonomous_strawberry_pedicel_harvesting
```

### Step 2: Create Conda Environment

```bash
conda create -n lerobot python=3.12 -y
conda activate lerobot
```

### Step 3: Install PyTorch for Jetson Orin

```bash
pip install torch --index-url https://download.pytorch.org/whl/cu118
```

Verify CUDA is available:

```bash
python -c "import torch; print('CUDA:', torch.cuda.is_available(), '| GPU:', torch.cuda.get_device_name(0))"
# Expected: CUDA: True | GPU: Orin
```

### Step 4: Install LeRobot

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot && pip install -e ".[so101]" && cd ..
```

### Step 5: Install Additional Dependencies

```bash
pip install huggingface_hub safetensors opencv-python pandas numpy scikit-learn matplotlib
```

### Step 6: Download Pre-trained Model

```bash
hf auth login   # paste your HuggingFace token

hf download nitindominicrai/act_strawberry_pedicel \
    --local-dir models/pretrained_model \
    --repo-type model

# Verify
ls models/pretrained_model/
# Expected: model.safetensors, config.json, train_config.json,
#           policy_preprocessor.json, policy_postprocessor.json, ...
```

---

## 3. Arm Calibration

Calibration maps the servo encoder values to physical joint angles. This must be done correctly as a bad calibration directly causes the arm to move to wrong positions during both teleoperation and autonomous inference.

```bash
sudo chmod 666 /dev/ttyACM0   # follower arm
sudo chmod 666 /dev/ttyACM1   # leader arm

# Calibrate follower arm
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm

# Calibrate leader arm
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm
```

**Critical calibration steps:**

| Step | What to do | Why it matters |
|---|---|---|
| Home position | Place arm fully upright and relaxed | Sets the zero reference |
| Gripper open | Open gripper as far as it physically goes | Defines the full open range |
| Gripper closed | Close gripper until fingers nearly touch | Defines the full close range |
| Joint range | Move each joint through its complete physical range | Maps encoder limits |

> **⚠️ Common mistake:** Not closing the gripper fully during calibration. The software maps 0–100% of the gripper range based on what you showed during calibration. If you only closed 80% of the way, the policy will only ever command up to that 80% and it will never fully grasp the pedicel.

Calibration files are saved to:
```
~/.cache/huggingface/lerobot/calibration/my_awesome_follower_arm.json
~/.cache/huggingface/lerobot/calibration/my_awesome_leader_arm.json
```

If you need to recalibrate, delete these files and run the calibration commands again.

---

## 4. Teleoperation and Verifying Setup

Before collecting data, verify teleoperation is working correctly. The follower arm should mirror the leader arm smoothly with no lag or erratic motion.

```bash
conda activate lerobot
cd ~/lerobot

lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ handeye: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30, rotation: 180}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```

This opens a Rerun viewer showing the live camera feed alongside joint positions. Check:
- Camera orientation is correct (berries visible, not upside down)
- Follower mirrors leader with <200ms lag
- Gripper opens and closes fully
- No servo overload warnings in terminal

> **✅ PRO TIP:** The `rotation: 180` flag is set because the wrist camera is physically mounted upside-down. This keeps the same 640×480 pixel dimensions while flipping the image right-side up. If your camera is mounted differently, adjust the rotation value (0, 90, 180, or 270) until the image looks correct in Rerun. Always use the same rotation value consistently throughout recording, training, and inference.

---

## 5. Dataset Collection

This is the most important step. The quality and consistency of your demonstrations directly determines policy performance. A noisy or inconsistent dataset produces a poor policy regardless of how long you train.

### 5.1 Recording Commands

```bash
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ handeye: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30, rotation: 180}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm \
    --dataset.repo_id=YOUR_HF_USERNAME/strawberry_harvest \
    --dataset.num_episodes=50 \
    --dataset.single_task="Pick the strawberry pedicel" \
    --dataset.reset_time_s=60 \
    --dataset.push_to_hub=true \
    --dataset.streaming_encoding=true \
    --dataset.encoder_threads=2
```

### 5.2 Episode Control

| Key | Action |
|---|---|
| `→` (right arrow) | Save current episode, enter reset phase |
| `←` (left arrow) | Discard current episode, re-record |
| `Esc` | Stop recording session |

> **⚠️ Important:** Press `→` only after completing a full demonstration. Press `Esc` only during the reset phase (between episodes), not during an active recording — pressing `Esc` mid-recording causes an empty episode crash.

### 5.3 Demonstration Protocol

The reach–grasp–pull sequence should be performed consistently across all episodes:

| Phase | Action | Duration |
|---|---|---|
| **Approach** | Move gripper toward pedicel from above | 3–5s |
| **Alignment** | Align gripper axis with pedicel stem | 2–3s |
| **Grasp** | Close gripper slowly around pedicel | 2–3s |
| **Pull** | Gentle upward pull to separate from calyx | 2–3s |
| **Retract** | Return arm to home position | 3–5s |

> **✅ PRO TIP:** Slow down significantly during the approach and grasp phases. Fast motions during contact produce blurry wrist camera frames and noisy joint readings — the policy learns from exactly what it sees. Consistent, deliberate demonstrations outperform fast ones every time.

### 5.4 Dataset Quality Checklist

Before uploading, verify your dataset:

```python
from lerobot.datasets.lerobot_dataset import LeRobotDataset

dataset = LeRobotDataset(repo_id="YOUR_HF_USERNAME/strawberry_harvest")
print(f"Episodes : {dataset.num_episodes}")
print(f"Frames   : {dataset.num_frames:,}")

sample = dataset[0]
print(f"Image    : {sample['observation.images.handeye'].shape}")
print(f"Action   : {sample['action'].shape}")
```

Expected output:
```
Episodes : 50+
Frames   : 60,000+
Image    : torch.Size([3, 480, 640])
Action   : torch.Size([6])
```

---

## 6. Training on HiPerGator (University of Florida)

I would recommend training the ACT Policy on GPU with amount of VRAM. I trained the policy on HiPerGator B200 GPU. 160K steps takes approximately 4-5 hours on a B200.

### 6.1 SSH and Environment Setup

```bash
# Windows PowerShell
ssh [your-username]@hpg.rc.ufl.edu

# On HiPerGator
source /path/to/your/virtual/environment/
```

### 6.2 Training Command

```bash
python -m lerobot.scripts.lerobot_train \
    --dataset.repo_id=YOUR_HF_USERNAME/strawberry_harvest \
    --policy.type=act \
    --output_dir=outputs/act_strawberry \
    --job_name=act_strawberry \
    --policy.device=cuda \
    --wandb.enable=false \
    --policy.push_to_hub=false \
    --steps=100000 \
    --batch_size=32 \
    --save_freq=20000 \
    --log_freq=200 \
    --dataset.video_backend=pyav \
    --dataset.use_imagenet_stats=false \
    --policy.vision_backbone=resnet50 \
    --policy.pretrained_backbone_weights=ResNet50_Weights.IMAGENET1K_V2
```

### 6.3 Key Hyperparameters

| Parameter | Value | Notes |
|---|---|---|
| `vision_backbone` | `resnet50` | Better features than default resnet18 for small targets |
| `pretrained_backbone_weights` | `ResNet50_Weights.IMAGENET1K_V2` | V2 weights give +5% better features |
| `batch_size` | `32` | I started facing issues with 64 batch size on B200 for some reason. Training got slower |
| `steps` | `100000` | Loss converges from ~2.84 to ~0.09 |
| `chunk_size` | `100` | 100 steps = 3.3s of committed action |
| `kl_weight` | `10.0` | Regularization for CVAE latent space |

### 6.4 Interpreting the Loss Curve

```
Initial loss  ~2.84  ← random policy
Step 10K      ~0.50  ← learning approach trajectory
Step 40K      ~0.15  ← KLD converging to zero
Step 100K     ~0.09  ← well converged
```

| Loss value | Interpretation |
|---|---|
| > 1.0 | Still learning basic motions |
| 0.3–1.0 | Partial convergence, trajectory visible |
| 0.1–0.3 | Good convergence for most joints |
| < 0.1 | Well converged and ready for deployment |

> **⚠️ Important:** Low training loss does not guarantee high success rate on the real arm. Generalization depends on demonstration diversity, not just loss convergence. 50 consistent demos typically converge well; 100+ demos improve real-world performance.

---

## 7. Converting to FP16 for Edge Deployment

FP16 reduces model size and could potentially improve inference speed on Jetson Orin from ~15 Hz to ~20 Hz. However, while I was implementing it, I didn't observe high frequency while inferencing. The script for conversion can be found at my [GitHub page](https://github.com/nitin-dominic/autonomous_strawberry_pedicel_harvesting/blob/main/scripts/convert_to_fp16.py).

```bash
conda activate lerobot
cd ~/lerobot

python scripts/convert_to_fp16.py
```

The conversion script saves the FP16 model to:
```
/path/to/your/local/drive/on/Jetson/
```

Verify:

```bash
python -c "
import sys; sys.path.insert(0, 'src')
from lerobot.policies.act.modeling_act import ACTPolicy
policy = ACTPolicy.from_pretrained(
    '/path/to/your/local/drive/on/Jetson/',
    local_files_only=True
)
import torch
policy.cuda().half()
dummy = torch.randn(1, 3, 480, 640, dtype=torch.float16).cuda()
with torch.inference_mode():
    _ = policy.model.backbone(dummy)
print('FP16 model verified')
"
```

---

## 8. Autonomous Inference on Jetson Orin

```bash
conda activate lerobot
cd ~/lerobot
sudo chmod 666 /dev/ttyACM0

lerobot-rollout \
    --strategy.type=base \
    --policy.path=/path/to/your/local/drive/on/Jetson/ \
    --policy.device=cuda \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras="{ handeye: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30, rotation: 180}}" \
    --task="Pick the strawberry pedicel" \
    --duration=45
```

The arm executes the learned reach–grasp–pull–retract sequence autonomously from wrist-camera input at 15–20 Hz.

> **✅ PRO TIP:** Use `--duration=45` for a 45-second episode. The arm will stop after this time regardless of task completion. For physical trials, note the outcome (success/partial/failure) manually after each run using the trial logging script in `scripts/log_trials.py`.

### Camera Configuration Reference

The camera config must exactly match between recording and inference:

| Step | width | height | rotation | Tensor shape |
|---|---|---|---|---|
| Recording | 640 | 480 | 180° | [480, 640, 3] |
| Inference | 640 | 480 | 180° | [480, 640, 3] |

A mismatch here causes the policy to receive spatially incorrect input and produce erratic arm motion without crashing — it is a silent failure mode.

---

## 9. Benchmarking Inference Speed

Run this on Jetson to measure actual inference speed:

```bash
conda activate lerobot
cd ~/lerobot

python scripts/benchmark_inference.py
```

Expected output on Jetson Orin AGX:

```
Method                      Speed       Hz    vs Baseline
────────────────────────────────────────────────────────
FP32 (baseline)            23.1ms     43Hz        1.00x
FP16                       20.0ms     50Hz        1.16x ← BEST
torch.compile              FAILED     (Python 3.12 not supported)
```

Full control loop (camera + inference + motor command) runs at **15–20 Hz** in practice — the camera capture and motor communication add ~30–40ms overhead beyond backbone inference alone.

---

## 10. Failure Analysis and Known Limitations

These are honest limitations from real-world testing:

**Primary failure mode: gripper misalignment:**
The arm consistently reaches the correct area but the gripper closes slightly off-axis from the pedicel stem. The 1.4–2.4 mm target leaves almost no margin for lateral error. This is attributed to limited demonstration diversity — the policy has not seen enough variation in approach angles to generalize robustly.

**Secondary failure mode: motor overload:**
Motor ID 5 (wrist roll) triggers overload protection when the gripper closes against the berry body rather than the pedicel stem. This causes the rollout to crash mid-episode. Wrap the disconnect method in a try/except block in `so_follower.py` to allow clean episode saving despite the overload.

**Hardware limitation: no cutting mechanism (not included in this work):**
The pincer end-effector grasps but cannot sever the pedicel. Full harvest requires a cutting or twisting end-effector — identified as a necessary hardware extension for this work.

| 🔴 Problem | 🔍 Likely Cause | ✅ Fix |
|---|---|---|
| Arm approaches wrong area | Camera resolution mismatch | Verify tensor shape matches training |
| Gripper closes but slips | Insufficient contact precision | Collect more demos at varied angles |
| Motor overload on disconnect | Servo protection triggered | Wrap disconnect in try/except |
| Inference too slow (<10 Hz) | Using FP32 | Convert to FP16 |
| Arm moves erratically | Camera config mismatch | Match rotation value exactly |
| Calibration ValueError | Servo at mechanical limit during calibration | Reset to home position, re-run calibrate |

---

## Enjoy Reading This Article?

Here are some more articles you might like to read next:

- [Autonomous Strawberry Picking Using a ROS2 Mobile Manipulator and YOLOv11-Based Depth-Aware Grasping](https://nitin-dominic.github.io/NR/blog/2026/ROS_Robotics_Strawberry/)

---

*© Copyright 2026 Nitin Rai. University of Florida — Agricultural and Biological Engineering.*
