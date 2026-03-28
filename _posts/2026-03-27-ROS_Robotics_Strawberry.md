---
layout: post
title: "From Perception-to-Actuation: A Complete Pipeline for Autonomous Strawberry Picking Using YOLOv11, Depth-Based Inverse Kinematics, and ROS2 Approach"
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

This write-up is for those working with the **HiWonder LanderPi** robot and want to implement **autonomous strawberry detection and picking** using computer vision and robotic arm control. However, the major focus of this tutorial is targeted towards **optimized arm control** and an end-effector **gripper** rather than vision-based detection.
The entire pipeline covers:

- Training a custom **YOLOv11** model on ripe/unripe strawberry classes (Very brief!)
- Converting the model to **OpenVINO IR format** for faster inference on Raspberry Pi (Very brief!)
- Deploying it on LanderPi running **ROS2 Humble** inside Docker (Detailed!)
- Using a **depth camera + inverse kinematics** to physically pick ripe strawberries (Detailed!)

> ⚠️ **Prerequisites** ⚠️
- Basic knowledge of Robot Operating System (ROS) and Computer Vision,
- Working with Linux-based OS,
- Must have followed primary instructions as stated within [Robotic Arm Control/LanderPi](https://wiki.hiwonder.com/projects/LanderPi/en/latest/),
- A virtual machine (VM) with Ubuntu 22.04 (Jammy) to visualize simulations using [Rviz](https://github.com/ros-visualization/rviz) (Optional!!!),
- A working knowledge of training vision models ([YOLO](https://github.com/ultralytics/ultralytics) in this case) using necessary hyperparameters,
- HiWonder LanderPi with Mecanum chassis,
- Aurora 930 depth camera,
- ROS2 Humble running inside Docker,
- The workspace lives at `/home/ubuntu/ros2_ws` within the container [1], and
- All commands must run inside the Docker container unless stated otherwise.

---

## 📋 Table of Contents

1. [Hardware and Software Requirements](#1-hardware-and-software-requirements)
2. [Training the Strawberry Detection Model](#2-training-the-strawberry-detection-model)
3. [Converting the Model for Deployment](#3-converting-the-model-for-deployment)
4. [Setting Up the ROS2 Package with Strawberry Pick Launch Files](#4-setting-up-the-ros2-package-with-strawberry-pick-launch-files)
5. [How the Picking Pipeline Works?](#5-how-the-picking-pipeline-works?)
6. [The Main Node](#6-the-main-node-strawberry_pick_ikpy)
7. [The Launch File](#7-the-launch-file-strawberry_pick_iklaunchpy)
8. [Running the Pipeline](#8-running-the-pipeline)
9. [Tuning the Arm for Accurate Picking](#9-tuning-the-arm-for-accurate-picking)
10. [Known Issues and Limitations](#10-known-issues-and-limitations)

---

## 1. Hardware and Software Requirements

<table style="width:100%; border-collapse:collapse; font-size:0.9em;">
  <thead>
    <tr style="background-color:#2d2d2d; color:white;">
      <th style="padding:8px 12px; text-align:left;">Component</th>
      <th style="padding:8px 12px; text-align:left;">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>🤖 Robot</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">HiWonder LanderPi with Mecanum chassis</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>📷 Camera</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Aurora 930 depth camera (RGB + depth)</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>💻 Compute</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Raspberry Pi 5 inside Docker container</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>🐧 OS</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Ubuntu 22.04, ROS2 Humble</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>🧠 ML Framework</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Ultralytics YOLOv11 + OpenVINO Runtime</td>
    </tr>
    <tr>
      <td style="padding:6px 12px;"><strong>🐍 Language</strong></td>
      <td style="padding:6px 12px;">Python 3.10</td>
    </tr>
  </tbody>
</table>
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
    data='strawberry_data.yaml',
    epochs=100,
    imgsz=640,
    batch=16,
    name='strawberry_yolo11'
)
```
---

## 3. Converting the Model for Deployment on LanderPi

The LanderPi's `yolov11_detect node` uses OpenVINO IR format `(.xml/.bin)` for
faster inference on the Raspberry Pi CPU strawberry_pick_ik.launch.py. The full conversion pipeline is:

```text
strawberry.pt  ──►  strawberry.onnx  ──►  strawberry.xml + strawberry.bin
```

##### Step 1: Export `.pt` to ONNX

```python
from ultralytics import YOLO
model = YOLO("strawberry.pt")
model.export(format="onnx", opset=12)
# Generates: strawberry.onnx
```

##### Step 2: Convert ONNX to OpenVINO IR

```python
# Using newer OpenVINO CLI (recommended)
ovc strawberry.onnx --output_model strawberry.xml

# Or using older Model Optimizer
mo --input_model strawberry.onnx --output_dir openvino_model/
```

This generates:

- `strawberry.xml` — model architecture
- `strawberry.bin` — model weights

##### Step 3: Place the Model Files

```console
cp strawberry.xml strawberry.bin MentorPi:/home/ubuntu/ros2_ws/src/yolov11_detect/models/
```
---

## 4. Setting Up the ROS2 Package with Strawberry Pick Launch Files

The strawberry picking node lives inside the example package under `rgbd_function/`. The workspace is automatically sourced via /source .zshrc → .robotrc every time you open a
new shell

```text
ros2_ws/src/example/example/rgbd_function/
├── __init__.py                    ← Required for Python module resolution
├── strawberry_pick_ik.py          ← Main picking node
└── strawberry_pick_ik.launch.py   ← Launch file
```

Entry Point in setup.py. This `setup.py` lives inside ros2_ws/src/. Use the command line below to edit this and add the example `strawberry_pick_ik` within this file. Doing this will add to the source file and then you can rebuild the packages.

```console
cd ros2_ws
ls
vim setup.py
```

Within the vim file, press `i` to insert the line, `'strawberry_pick_ik = example.rgbd_function.strawberry_pick_ik:main'`. Once inserted, press ESC and `:wq` to save and exit. If you do not prefer vim comamnd, alternatively, you can also do `gedit` and edit the file. However, please double-check it. I am not sure if this will work.

```vim
'console_scripts': [
    # ... existing entries ...
    'strawberry_pick_ik = example.rgbd_function.strawberry_pick_ik:main',
],
```
Just in case for whatever reasons, if `__init__.py` file does not exist within the `/rgbd_function` folder, create one using the command below. This step is important because when building a package, python needs to treat ````example.rgbd_function```` as a package, not just a folder. The ``__init__.py`` file is what tells Python *this directory* is a Python package that can be imported. Without it, ROS2 will try to load your node, Python will see ``rgbd_function/`` as just a plain folder and wil throw an error.

```console
touch ~/ros2_ws/src/example/example/rgbd_function/__init__.py
```
Finally, head-on to building a package using the command line below. The ``--symlink-install`` flag is particularly useful during development. Since it creates symlinks from the install directory directly to your source files command, any changes you make to an existing ``.py`` file like ``strawberry_pick_ik.py`` are instantly reflected without needing to rebuild. You only need to rebuild when adding new files or new entry points to ``setup.py``.

```console
cd ~/ros2_ws
colcon build --event-handlers console_direct+ --cmake-args -DCMAKE_BUILD_TYPE=Release --symlink-install --packages-select example
```
---

## 5. How the Picking Pipeline Works?

The full autonomous pipeline runs as follows:

<table style="width:100%; border-collapse:collapse; font-size:0.9em; margin-bottom:1.5em;">
  <thead>
    <tr style="background-color:#2d2d2d; color:white;">
      <th style="padding:8px 12px; text-align:left;">Stage</th>
      <th style="padding:8px 12px; text-align:left;">What Happens</th>
      <th style="padding:8px 12px; text-align:left;">ROS2 Topic / Service</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>🧠 YOLO Detection</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">YOLOv11 detects ripe and unripe strawberries in every frame</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>/yolo_node/object_detect</code></td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>📷 Image Sync</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">RGB, depth, and camera_info are time-synchronized</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>/ascamera/camera_publisher/rgb0/image</code></td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>🎯 PID Tracking</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Camera servos adjust to center the ripe berry in frame using PID control</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>/servo_controller</code></td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>⏱️ Stability Check</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Waits 5 seconds of stable PID tracking before triggering the grab</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Internal timer</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>📐 Depth + IK</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Depth ROI averaged → pixel + depth → 3D camera coords → world coords via hand-eye matrix</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>/kinematics/set_pose_target</code></td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>🦾 Grab</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Arm moves to computed 3D position and gripper closes around the berry</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>/servo_controller</code></td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>⬆️ Lift</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Arm lifts berry slightly upward before moving to safe retract position</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>/servo_controller</code></td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><strong>📦 Place</strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Arm moves to placement position on the left side and gripper opens to release</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>/servo_controller</code></td>
    </tr>
    <tr>
      <td style="padding:6px 12px;"><strong>🏠 Return Home</strong></td>
      <td style="padding:6px 12px;">Arm returns to initial scanning position and tracker resets for next berry</td>
      <td style="padding:6px 12px;"><code>/servo_controller</code></td>
    </tr>
  </tbody>
</table>

### Key Tuning Parameters

<table style="width:100%; border-collapse:collapse; font-size:0.9em; margin-bottom:1.5em;">
  <thead>
    <tr style="background-color:#2d2d2d; color:white;">
      <th style="padding:8px 12px; text-align:left;">Parameter</th>
      <th style="padding:8px 12px; text-align:left;">Location</th>
      <th style="padding:8px 12px; text-align:left;">Default</th>
      <th style="padding:8px 12px; text-align:left;">What it controls</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>conf</code></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">launch file [5]</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>0.45</code></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">YOLO confidence threshold</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>position[2] -= X</code></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">strawberry_pick_ik.py [6]</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>0.02</code></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">How low the arm reaches to grab</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Stability timer</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">strawberry_pick_ik.py [6]</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>5 sec</code></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Wait time before triggering grab</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Depth ROI</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">strawberry_pick_ik.py [6]</td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;"><code>24×24 px</code></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">Area used for depth averaging</td>
    </tr>
    <tr>
      <td style="padding:6px 12px;">PID threshold</td>
      <td style="padding:6px 12px;">strawberry_pick_ik.py [6]</td>
      <td style="padding:6px 12px;"><code>0.01</code></td>
      <td style="padding:6px 12px;">Sensitivity of camera centering adjustments</td>
    </tr>
  </tbody>
</table>

---

## 6. The Main Node: ```strawberry_pick_ik.py```

The node uses a custom YoloTracker class that replaces traditional HSV color detection with YOLO bounding box centers ``strawberry_pick_ik.py``. It subscribes to /yolo_node/object_detect and uses PID controllers to center the camera on the best detected ripe strawberry. Within the `strawberry_pick_ik.py`, look for these lines (below) to understand the code structure. 

YoloTracker Class

```python
class YoloTracker:
    """Uses YOLO bounding box center for PID tracking."""

    def __init__(self, target_class='ripe'):
        self.target_class = target_class
        self.pid_yaw = pid.PID(20.5, 1.0, 1.2)
        self.pid_pitch = pid.PID(20.5, 1.0, 1.2)
        self.yaw = 500
        self.pitch = 150
        self.center = None
        self.detected = False

    def update_detection(self, objects):
        best = None
        for obj in objects:
            if obj.class_name == self.target_class:
                cx = (obj.box[0] + obj.box[2]) / 2.0
                cy = (obj.box[1] + obj.box[3]) / 2.0
                if best is None or obj.score > best[3]:
                    best = (cx, cy,
                            max(abs(obj.box[2]-obj.box[0]),
                                abs(obj.box[3]-obj.box[1]))/2.0,
                            obj.score)
        if best is not None:
            self.center = (best[0], best[1])
            self.detected = True
        else:
            self.center = None
            self.detected = False
```

Stability Check and Depth-to-3D Conversion

```python
# Stability check: wait 5 seconds of settled tracking before grabbing. Make sure this deplay between detection and grabbing is at least 5 secs. I faced a lot of problem with values below 3. The strawberry was being detected but the arm did not get time to settle-in.
if abs(self.last_pitch_yaw[0] - p_y[0]) < 3 \
        and abs(self.last_pitch_yaw[1] - p_y[1]) < 3:
    if time.time() - self.stamp > 5 and not self.moving:

        # Enlarged 24x24 ROI for stable depth reading
        roi = [
            max(0, int(center_y) - 12), min(h, int(center_y) + 12),
            max(0, int(center_x) - 12), min(w, int(center_x) + 12),
        ]
        roi_distance = depth_image[roi[0]:roi[1], roi[2]:roi[3]]
        valid_depths = roi_distance[
            np.logical_and(roi_distance > 0, roi_distance < 10000)]
        dist = round(float(np.mean(valid_depths) / 1000.0), 3)

        dist += 0.015  # Object radius compensation
        dist += 0.015  # Error compensation

        # Convert pixel + depth → 3D camera coordinates
        K = depth_camera_info.k
        position = depth_pixel_to_camera(
            (center_x, center_y), dist, (K[0], K[4], K[2], K[5]))

        # RGB-to-depth camera offset compensation
        position[0] -= 0.01
        position[1] -= 0.02
        position[2] -= 0.02  # ← Tune this if arm grabs above or below berry

        # Transform to world coordinates via hand-eye calibration matrix
        pose_end = np.matmul(
            self.hand2cam_tf_matrix,
            common.xyz_euler_to_mat(position, (0, 0, 0)))
        world_pose = np.matmul(self.endpoint, pose_end)
        pose_t, _ = common.mat_to_xyz_euler(world_pose)

        self.moving = True
        threading.Thread(target=self.pick, args=(pose_t,)).start()
```
