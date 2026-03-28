---
layout: post
title: "Autonomous Strawberry Detection and Picking Using YOLOv11, ROS2, and HiWonder LanderPi"
date: 2026-03-27 09:00:00-0400
description: "A complete guide to training a custom YOLOv11 model, converting it to OpenVINO format, and deploying it on a LanderPi robot for autonomous strawberry detection and picking using a depth camera and inverse kinematics."
tags: ros2 robotics yolo deep-learning raspberry-pi agriculture computer-vision
categories: robotics
thumbnail: assets/img/strawberry_pick.png

---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/strawberry_pick.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    HiWonder LanderPi autonomously detecting and picking a ripe strawberry using YOLOv11 and depth-based inverse kinematics.
</div>

---

## 🍓 Overview

This write-up is for those working with the **HiWonder LanderPi** robot who want to implement
**autonomous strawberry detection and picking** using computer vision and robotic arm control.
The entire pipeline covers:

- Training a custom **YOLOv11** model on ripe/unripe strawberry classes
- Converting the model to **OpenVINO IR format** for faster inference on Raspberry Pi
- Deploying it on LanderPi running **ROS2 Humble** inside Docker
- Using a **depth camera + inverse kinematics** to physically pick ripe strawberries

> ⚠️ **Prerequisites** ⚠️
- Basic knowledge of Robot Operating System (ROS) and Computer Vision,
- Working with Linux-based OS,
- Must have followed primary instructions as stated within [Robotic Arm Control/LanderPi](https://wiki.hiwonder.com/projects/LanderPi/en/latest/),
- A virtual machine (VM) with Ubuntu 22.04 (Jammy) to visualize simulations using [Rviz](https://github.com/ros-visualization/rviz) (Optional!!!),
- A working knowledge of training vision models ([YOLO](https://github.com/ultralytics/ultralytics) in this case) using necessary hyperparameters,
- HiWonder LanderPi with Mecanum chassis,
- Aurora 930 depth camera,
- ROS2 Humble running inside Docker,
- The workspace lives at `/home/ubuntu/ros2_ws` within the container [1],
- All commands must run inside the Docker container unless stated otherwise,

---

## 📋 Table of Contents

1. [Hardware and Software Requirements](#1-hardware-and-software-requirements)
2. [Training the Strawberry Detection Model](#2-training-the-strawberry-detection-model)
3. [Converting the Model for Deployment](#3-converting-the-model-for-deployment)
4. [Setting Up the ROS2 Package](#4-setting-up-the-ros2-package)
5. [How the Picking Pipeline Works](#5-how-the-picking-pipeline-works)
6. [The Main Node](#6-the-main-node-strawberry_pick_ikpy)
7. [The Launch File](#7-the-launch-file-strawberry_pick_iklaunchpy)
8. [Running the Pipeline](#8-running-the-pipeline)
9. [Tuning the Arm for Accurate Picking](#9-tuning-the-arm-for-accurate-picking)
10. [Known Issues and Limitations](#10-known-issues-and-limitations)

---

## 1. Hardware and Software Requirements

| Component | Details |
|---|---|
| **Robot** | HiWonder LanderPi with Mecanum chassis [2] |
| **Camera** | Aurora 930 depth camera (RGB + depth) [2] |
| **Compute** | Raspberry Pi 5 inside Docker container [1] |
| **OS** | Ubuntu 22.04, ROS2 Humble [1] |
| **ML Framework** | Ultralytics YOLOv11 + OpenVINO Runtime [5] |
| **Language** | Python 3.10 |

---

## 2. Training the Strawberry Detection Model

> 🕐 **This is the most important and time-consuming part.** The quality of your dataset
> directly determines how well the robot detects strawberries. However, to save time, I have provided `strawberry.pt` file which you'll have to move to a particular destination in order to invoke detection which will then trigger picking nodes of the robotic arm.
> I have discussed this in [5](#5-how-the-picking-pipeline-works).

I trained a custom YOLOv11n (nano for Raspberry Pi) model with two classes: `ripe` and `unripe` uisng the dataset from [Roboflow](https://universe.roboflow.com/eric-z0ptd/strawberry_picking_2). However, within the repo that you'll clone, I have already provided `strawberry.pt` file to get started. I trained my own model with custom hyperparameters. With the interest of space within this `Blog`, I am not going explain in detail on how to train a YOLO-based vision model for detection purpose. If you encounter any issues, please reach out to me or pull an issue.

Your `strawberry_data.yaml` should look like this. Make sure when you're training, this `.yaml` file lives inside the main dataset folder where `test`, `train`, and `'valid` folders are.

```yaml
train: ../train/images
val: ../valid/images
test: ../test/images

nc: 2
names: ['ripe', 'unripe']

roboflow:
  workspace: eric-z0ptd
  project: strawberry_picking_2
  version: 1
  license: CC BY 4.0
  url: https://universe.roboflow.com/eric-z0ptd/strawberry_picking_2/dataset/1
```

```python
from ultralytics import YOLO
model = YOLO('yolo11n.pt')  # Start from nano pretrained weights
model.train(
    data='data.yaml',
    epochs=100,
    imgsz=640,
    batch=16,
    name='strawberry_yolo11'
)
```

## 3. Converting the Model for Deployment on LanderPi

The LanderPi's `yolov11_detect node` uses OpenVINO IR format `(.xml/.bin)` for
faster inference on the Raspberry Pi CPU strawberry_pick_ik.launch.py. The full conversion pipeline is:

```text
strawberry.pt  ──►  strawberry.onnx  ──►  strawberry.xml + strawberry.bin
```

### Step 1: Export `.pt` to ONNX

```python
from ultralytics import YOLO
model = YOLO("strawberry.pt")
model.export(format="onnx", opset=12)
# Generates: strawberry.onnx
```

### Step 2: Convert ONNX to OpenVINO IR

```python
# Using newer OpenVINO CLI (recommended)
ovc strawberry.onnx --output_model strawberry.xml

# Or using older Model Optimizer
mo --input_model strawberry.onnx --output_dir openvino_model/
```

This generates:

> `strawberry.xml` — model architecture
> `strawberry.bin` — model weights

### Step 3: Place the Model Files

```console
cp strawberry.xml strawberry.bin MentorPi:/home/ubuntu/ros2_ws/src/yolov11_detect/models/
```
## 4. Setting Up the ROS2 Package

The strawberry picking node lives inside the example package under `rgbd_function/`. The workspace is automatically sourced via /source .zshrc → .robotrc every time you open a
new shell 
.robotrc +1
 .
