---
layout: post
title: "Autonomous Strawberry Picking Using a ROS2 Mobile Manipulator and YOLOv11-Based Depth-Aware Grasping"
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
    A screenshot of HiWonder's LanderPi autonomously detecting and picking a 'ripe' strawberry using YOLOv11 and depth-based inverse kinematics. It consists of three parts within the video: (a) the real-time feed from the depth-camera detecting strawberry, (b) the middle screen with a simulation video on Rviz, and (c) real footage picking strawberry using the end-effector gripper.
</div>

---

## 🍓 🤖 Background 

Small robots (or mobile manipulators) are extremely helpful in settings that are compact or limited with space, such as greenhouses. Often, strawberries grown in greenhouses are harvested manually, and that is exactly where these small robots could make a real difference. Imagine multiple of these working in coordination (like a swarm), moving autonomously through narrow greenhouse rows, identifying only the berries that are ripe, and picking them one by one without disturbing the plant or the fruit around it. No fatigue, no missed berry, no damage!

This work imitated, in a small way, a situation in which ripe strawberries could be detected and picked autonomously and the whole process could be automated. Using YOLOv11 for visual detection 
`(strawberry_pick_ik.launch.py)`, a depth camera for 3D depth estimation, and inverse kinematics to physically actuate the robotic arm `(strawberry_pick_ik.py)`, the HiWonder LanderPi demonstrated that even a compact, low-cost mobile manipulator running ROS2 Humble inside Docker on a Raspberry Pi 5 can perform a task as nuanced and delicate as selective fruit picking.

The broader implication is not just about strawberries. It is about demonstrating that precision agriculture, the kind that requires *selective*, *gentle*, and *intelligent* interaction with individual plants, no longer has to depend entirely on human hands.

## 🍓 Overview

This write-up is for those working with a **HiWonder LanderPi** robot and want to implement **autonomous strawberry detection and picking** using computer vision and robotic arm control. However, the major focus of this tutorial is targeted towards **optimized arm control** and an end-effector **gripper** rather than vision-based detection. If you already know ROS and computer vision, you can clone this repo [LanderPi](https://github.com/nitin-dominic/LanderPi.git) on your RaspberryPi as I have already added all the configuration files along with the trained strawberry model architecture + relevant `.py` files: `strawberry_pick_ik.py` and `strawberry_pick_ik.launch.py`, to jump-start detection and picking. 

All you will have to do is to simply move the relevant files into respective docker container folders and perform build in order to add strawberry detection and picking package to the ROS2 `setup.py` file. This process won't take more than 15-20 mins provided you know working with Linux-based system. If you want to learn step-by-step process, then start following the steps as mentioned below. 

<div style="background-color:#d4edda; border-left:6px solid #28a745; 
padding:12px 16px; border-radius:4px; margin:1em 0; color:#000000;"> <strong>✅</strong> PRO TIP: Remember, following the steps below could be painful and extremely slow sometimes / some days. Be consistent as this learning curve could lead to very fruitful results once you see the robot picking strawberries. It happened with me!
</div>

The entire pipeline covers:

- Training a custom **YOLOv11** model on ripe/unripe strawberry classes (Very brief!)
- Converting the model to **OpenVINO IR format** for faster inference on Raspberry Pi (Very brief!)
- Deploying it on LanderPi running **ROS2 Humble** inside Docker (Detailed!)
- Using a **depth camera + inverse kinematics** to physically pick ripe strawberries (Detailed!)


<div style="background-color:#fff3cd; border-left:6px solid #ffc107;
padding:12px 16px; border-radius:4px; margin:1em 0;">

  <strong style="color:#000000; font-size:1.1em;">⚠️ Prerequisites ⚠️</strong>

  <ul style="color:#000000; margin-top:8px; margin-bottom:0;">
    <li style="color:#000000;">• Basic knowledge of Robot Operating System (ROS) and Computer Vision</li>
    <li style="color:#000000;">• Working with Linux-based OS</li>
    <li style="color:#000000;">• Must have followed primary instructions as stated within
        <a href="https://wiki.hiwonder.com/projects/LanderPi/en/latest/" 
        style="color:#0056b3;"> Robotic Arm Control/LanderPi</a></li>
    <li style="color:#000000;">• A virtual machine (VM) with Ubuntu 22.04 (Jammy) to visualize 
        simulations using <a href="https://github.com/ros-visualization/rviz" 
        style="color:#0056b3;">RViz</a> 
        <strong style="color:#000000;">(Optional)</strong></li>
    <li style="color:#000000;">• A working knowledge of training vision models 
        (<a href="https://github.com/ultralytics/ultralytics" 
        style="color:#0056b3;"> YOLO</a> in this case) 
        using necessary hyperparameters</li>
    <li style="color:#000000;">• HiWonder LanderPi with Mecanum chassis [2]</li>
    <li style="color:#000000;">• Aurora 930 depth camera [2]</li>
    <li style="color:#000000;">• ROS2 Humble running inside Docker [1]</li>
    <li style="color:#000000;">• The workspace lives at 
        <code style="background-color:#e8e8e8; color:#000000; 
        padding:2px 4px; border-radius:3px;">/home/ubuntu/ros2_ws</code> 
        within the container [1]</li>
    <li style="color:#000000;">• All commands must run inside the Docker container 
        unless stated otherwise</li>
  </ul>
</div>
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

<div style="background-color:#cce5ff; border-left:6px solid #004085; 
padding:12px 16px; border-radius:4px; margin:1em 0; color:#000000;"> <strong>ℹ️</strong>Note: This is the most important and time-consuming part. The quality of your dataset directly determines how well the robot detects strawberries. However, to save time, I have provided `strawberry.pt` file which you'll have to move to a particular destination in order to invoke detection which will then trigger picking nodes of the robotic arm. I have discussed this in [5].
</div>

I trained a custom YOLOv11n (nano for Raspberry Pi) model with two classes: `ripe` and `unripe` uisng the dataset from [Roboflow](https://universe.roboflow.com/eric-z0ptd/strawberry_picking_2). However, within the repo that you'll clone, I have already provided `strawberry.pt` file to get started. I trained my own model with custom hyperparameters. With the interest of space within this `Blog`, I am not going explain in detail on how to train a YOLO-based vision model for detection purpose. If you encounter any issues, please reach out to me or pull an issue.

Your `strawberry_data.yaml` should look like this (below). Make sure when you are training your own YOLOv11n model, this `*.yaml` file lives inside the main dataset folder where `test`, `train`, and `valid` folders are present.

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

Once your model is trained, you will see `strawberry.pt` (or whatever name you trained your model with) is saved inside the cusotm path. Now this step completely depends on where you trained your model. There are two options: (a) if you trained the model on the Pi, the `.pt` file stays on your Pi based on whatever path you gave, and (b) if you trained it on your desktop with better GPU VRAM (it is recomemnded since Pi will be slow), then you will have to move all the relevant files from the desktop to the Pi. In this case, I have already provided these files on my forked [repo](https://github.com/nitin-dominic/LanderPi/tree/main). But if you trained the model, then just run the command below in order to generate `.xml` and `.bin` files using your `'strawberry.pt.` file. The LanderPi's `yolov11_detect node` uses OpenVINO IR format `(.xml/.bin)` for faster inference on the Raspberry Pi CPU strawberry_pick_ik.launch.py. So, the next following step is to convert The full conversion pipeline is:

```text
strawberry.pt  ──►  strawberry.xml + strawberry.bin
```

##### Step 1: Export `.pt` to `.xml` and `.bin` files

```console
pip install openvino-dev --break-system-packages #install this package if not already 
```

```python
# On the Pi (or your VM) 
# Export your .pt to OpenVINO format
from ultralytics import YOLO
yolo export model=/path/to/strawberry.pt format=openvino imgsz=640

# This creates a folder with:
#   strawberry_openvino_model/
#     strawberry.xml
#     strawberry.bin
```

This creates a folder `strawberry_openvino_model/` and two files within it:

- `strawberry.xml` — model architecture
- `strawberry.bin` — model weights

##### Step 2: Place the Model Files Within the Docker Container on the Pi

```console
# Copy the .xml and .bin to the models directory

cp path/to/your/strawberry.xml MentorPi:/home/ubuntu/ros2_ws/src/yolov11_detect/models/
cp path/to/your/strawberry.bin MentorPi:/home/ubuntu/ros2_ws/src/yolov11_detect/models/

# Check if both the files have been successfully copied.

cd ros2_ws/src/yolov11_detect/models/
ls
```
---

## 4. Setting Up the ROS2 Package with Strawberry Pick Launch Files

Before getting into this section, let's understand why does ROS2 need a `setup.py` entry point? It will be worth understanding why this is necessary considering you plan on doing custom work with LanderPi beyond just this tutorial. In ROS2, when you create a Python-based package, the `setup.py` file is the bridge
between your Python source code (in this case the strawberry `.py` files) and the ROS2 build system (colcon). Without this entry, ROS2 has no idea your node exists even if the `.py` file is physically sitting in the right folder. This is why running `ros2 launch example strawberry_pick_ik.launch.py` without the entry point will throw:

```text
PackageNotFoundError: No package metadata was found for example
```
##### Step 1: Add the Entry Point to `setup.py`

The `setup.py` file lives at `~/ros2_ws/src/example/setup.py`. Open it either using `vim` or `gedit`. For me `gedit` did not work for the first time and threw errors. So, I would recommend using `vim`. 

```console
cd ~/ros2_ws/src/example
vim setup.py
```

<div style="background-color:#fff3cd; border-left:6px solid #ffc107;
padding:12px 16px; border-radius:4px; margin:1em 0; color:#000000;">
<strong>⚠️</strong>vim quick reference: Press <code>i</code> to enter insert mode, make your edit, then press <code>ESC</code> followed by <code>:wq</code> to save and exit. If you make a mistake, press <code>ESC</code> then <code>:q!</code> to exit without saving.
</div>

Once inside the `setup.py` using `vim`, console_scripts section and add your entry point:

```vim
'console_scripts': [
    # ... existing entries ...
    'strawberry_pick_ik = example.rgbd_function.strawberry_pick_ik:main',
],
```

<div style="background-color:#cce5ff; border-left:6px solid #004085;
padding:12px 16px; border-radius:4px; margin:1em 0; color:#000000;">
<strong>ℹ️</strong>Note: The existing entries in <code>console_scripts</code> are all the other LanderPi nodes — color detection, hand tracking, navigation transport, etc [4]. Do not remove or modify those lines. Just add your new line at the end of the list, making sure the previous line ends with a comma.
</div>

##### Step 2: Create `__init__.py` if It Does Not Exist

This step is important and easy to miss. Check if `__init__.py` already exists. Just in case for whatever reasons, if `__init__.py` file does not exist within the `/rgbd_function` folder, create one using the command below. 

```console
touch ~/ros2_ws/src/example/example/rgbd_function/__init__.py
```

So, why is this step important? When `colcon` builds the package, Python needs to treat `example.rgbd_function` as a proper importable package, not just a filesystem folder. The `__init__.py` file, even though it is completely empty, is the signal Python uses to make this distinction. Without it, when ROS2 tries to resolve your entry point `example.rgbd_function.strawberry_pick_ik:main`, Python will look for a package called `rgbd_function` inside example, finding only a folder with no `__init__.py`, and throw:

```text
ModuleNotFoundError: No module named 'example.rgbd_function'
```
##### Step 3: Build the ROS2 Package

Now rebuild the example package so `colcon` picks up your new entry point and registers it in the install directory. Use the code below.

```console
cd ~/ros2_ws
colcon build --event-handlers console_direct+ --cmake-args -DCMAKE_BUILD_TYPE=Release --symlink-install --packages-select example
```

Here is what flag means: 

<table style="width:100%; border-collapse:collapse; font-size:0.9em;">
  <thead>
    <tr style="background-color:#2d2d2d; color:white;">
      <th style="padding:8px 12px; text-align:left;">Flag</th>
      <th style="padding:8px 12px; text-align:left;">Meaning</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444; color:#000000;">
        <strong><code>colcon build</code></strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">
        The ROS2 build command that compiles all packages in the workspace [4]</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444; color:#000000;">
        <strong><code>--event-handlers console_direct+</code></strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">
        Prints the full build output directly to the terminal in real time — 
        useful for seeing errors immediately instead of buffering them [4]</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444; color:#000000;">
        <strong><code>--cmake-args -DCMAKE_BUILD_TYPE=Release</code></strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">
        Tells CMake to build in <strong>Release mode</strong>, enabling compiler 
        optimizations for faster and more efficient runtime performance 
        compared to Debug mode [4]</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; border-bottom:1px solid #444; color:#000000;">
        <strong><code>--symlink-install</code></strong></td>
      <td style="padding:6px 12px; border-bottom:1px solid #444;">
        Instead of <strong>copying</strong> files into the install directory, 
        creates <strong>symbolic links</strong> back to your source files [4]. 
        Any edits to existing <code>.py</code> files like 
        <code>strawberry_pick_ik.py</code> are instantly reflected 
        without rebuilding [6]</td>
    </tr>
    <tr>
      <td style="padding:6px 12px; color:#000000;">
        <strong><code>--packages-select example</code></strong></td>
      <td style="padding:6px 12px; color:#000000;">
        Builds <strong>only</strong> the <code>example</code> package instead 
        of rebuilding the entire workspace, much faster during 
        development [4]</td>
    </tr>
  </tbody>
</table>

<div style="background-color:#d4edda; border-left:6px solid #28a745; 
padding:12px 16px; border-radius:4px; margin:1em 0; color:#000000;">
  <strong style="color:#000000;">✅ Pro tip — The symlink advantage:</strong> 
  The <code style="background-color:#c3e6cb; color:#000000; padding:2px 4px; 
  border-radius:3px;">--symlink-install</code> flag [4] is particularly useful 
  during development. Since it creates symlinks from the install directory directly 
  back to your source files, any changes you make to an existing 
  <code style="background-color:#c3e6cb; color:#000000; padding:2px 4px; 
  border-radius:3px;">.py</code> file like 
  <code style="background-color:#c3e6cb; color:#000000; padding:2px 4px; 
  border-radius:3px;">strawberry_pick_ik.py</code> [6] are 
  <strong style="color:#000000;">instantly reflected</strong> without needing 
  to rebuild. You only need to rebuild when adding new files or new entry points 
  to <code style="background-color:#c3e6cb; color:#000000; padding:2px 4px; 
  border-radius:3px;">setup.py</code>.
</div>

A successful build will show:

```text

Finished <<< example [Xs]
Summary: 1 package finished [Xs]
```

The `stderr` warnings about `setuptools` deprecation are harmless and can be ignored.

##### Step 4: Verify the Build

After building, confirm your node was installed correctly:

```console
ls ~/ros2_ws/install/example/lib/example/strawberry_pick_ik
```
And verify the `main()` function is present in your source file:

```console
grep -n "def main" ~/ros2_ws/src/example/example/rgbd_function/strawberry_pick_ik.py
```
Both should return valid output. If `grep` returns nothing, your source file is missing the `main()` function and the node will fail to launch. Remove the old log files and try to rebuild the package. 

##### Step 5: Open a Fresh Terminal and Launch

Open a fresh terminal inside the Docker container. Like I said before, everything is sourced already. However, you can source again using `source ~/.zshrc`. IT IS TIME TO LAUNCH AND SEE THE ROBOT IN ACTION!!!!!!!!! 

```console
ros2 launch example strawberry_pick_ik.launch.py
```

<div style="background-color:#ffcccc; border-left:6px solid #cc0000;
padding:12px 16px; border-radius:4px; margin:1em 0; color:#000000;">
<strong>⛔ </strong>Important: Always open a <strong>fresh terminal</strong> after
building. Reusing the same terminal keeps stale environment variables cached from
before the build, which can cause sourcing errors even if the build succeeded. I faced this issue and it took me a while to understand.
</div>

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

##### Key Tuning Parameters

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

## 7. The Launch Node: ```strawberry_pick_ik.launch.py```

The launch file starts all required nodes together in one command: `strawberry_pick_ik.launch.py`

```python
# YOLO detection node — loads strawberry.xml OpenVINO model
yolov11_node = Node(
    package='yolov11_detect',
    executable='yolov11_node',
    output='screen',
    parameters=[
        {'classes': ['ripe', 'unripe']},
        {'model': 'strawberry', 'conf': conf},
        {'start': True},
    ]
)

# Strawberry IK picking node
strawberry_pick_ik_node = Node(
    package='example',
    executable='strawberry_pick_ik',
    output='screen',
    parameters=[{'start': start}],
)
```
All nodes launched together:

- 📷 Depth camera (RGB + depth + camera_info)
- 🎮 Servo controller manager
- 🦾 Kinematics IK solver
- 🧠 YOLOv11 detection node
- 🍓 Strawberry pick IK node

## 8. Running the Pipeline
Open a terminal inside the Docker container. Before running any ros2 nodes, ensure running, `~/.stop_ros.sh` to stop any auto-start services as it can intervene with the oother nodes and may delay decision-making. You can also try to source it using, `source ~/.bashrc` to source again. However,to avoid any issues, make sure you open a new docker terminal so it is sourced automatically and you can run ros2 nodes successfully! 

```console
ros2 launch example strawberry_pick_ik.launch.py
```
