---
layout: post
title: Autonomous Strawberry Detection and Picking Using YOLOv11, ROS2, and HiWonder LanderPi
date: 2026-03-27 09:00:00-0400
description:
tags: ros2 robotics yolo deep-learning raspberry-pi agriculture
categories:
---

This write-up is for those who are working with the HiWonder LanderPi robot and want to
implement autonomous strawberry detection and picking using computer vision and robotic arm
control. Although deep learning and ROS2 can be used independently, I will explain how to
integrate both on the LanderPi platform running ROS2 Humble inside a Docker container on a
Raspberry Pi 5. The entire pipeline covers training a custom YOLOv11 model, converting it
to OpenVINO format for faster inference, and deploying it on the robot to detect and
physically pick ripe strawberries using a depth camera and inverse kinematics.

> **Note:** This tutorial assumes you have a HiWonder LanderPi robot with a Mecanum chassis,
> a depth camera (Aurora 930), and ROS2 Humble running inside Docker. The workspace is at
> `/home/ubuntu/ros2_ws` inside the container [1]. All commands should be run inside the
> Docker container unless stated otherwise.

---

## 1. Hardware and Software Requirements

**Hardware:**
- HiWonder LanderPi with Mecanum chassis [2]
- Aurora 930 depth camera (RGB + depth) [2]
- Robotic arm with servo controller
- Raspberry Pi 5

**Software:**
- ROS2 Humble (inside Docker container) [1]
- Python 3.10
- Ultralytics YOLOv11
- OpenVINO Runtime
- OpenCV

---

## 2. Training the Strawberry Detection Model

This is the most important and time-consuming part. The quality of your dataset directly
determines how well the robot detects strawberries. I trained a custom YOLOv11 model with
two classes: `ripe` and `unripe`.

### a. Collecting and Labeling Data

Collect images of strawberries in different lighting conditions, angles, and backgrounds.
The more diverse your dataset, the better the model performs. I used
[Roboflow](https://roboflow.com) to annotate images and export them in YOLO format.

Your `data.yaml` should look like this:

```yaml
train: /path/to/train/images
val: /path/to/val/images
nc: 2
names: ['ripe', 'unripe']

from ultralytics import YOLO

model = YOLO('yolo11n.pt')  # Start from nano pretrained weights
model.train(
    data='data.yaml',
    epochs=100,
    imgsz=640,
    batch=16,
    name='strawberry_yolo11'
)
