---
layout: post
title: "Generating Crop Disease Images Using Prompt Engineering with Stable Diffusion Architecture"
date: 2026-01-16 09:00:00-0400
description: "A complete guide to generating synthetic crop disease images using Stable Diffusion and prompt engineering on the University of Florida's HiPerGator HPC system."
tags: stable-diffusion prompt-engineering deep-learning agriculture computer-vision hipergator generative-ai
categories: generative-ai
thumbnail: assets/img/synthetic.png
---

<div class="row">
<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="assets/img/synthetic_images.png" title="example image" class="img-fluid rounded z-depth-1" style="background-color: white; padding: 8px;" %}
</div>
</div>
<div class="caption">
    Comparison of <strong>real</strong> vs. <strong>synthetic</strong> watermelon images. Synthetic images were generated using a <strong>text-to-image Stable Diffusion</strong> architecture to augment training data for classification, detection, or segmentation tasks.
</div>

## Overview

Acquiring labeled images of crop diseases is one of the biggest bottlenecks in agricultural AI research. Field data is scarce, seasonal, and expensive to annotate. **Stable Diffusion** offers a powerful alternative: generating photorealistic, diverse, and labeled synthetic images of diseased crops directly from text prompts, at scale, on GPU hardware.

This guide walks you through setting up a HiPerGator environment, installing Stable Diffusion, and using **prompt engineering** to generate high-quality crop disease images for training object detection or classification models.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Setting Up Your Conda Environment on HiPerGator](#2-setting-up-your-conda-environment-on-hipergator)
3. [Installing Stable Diffusion Libraries](#3-installing-stable-diffusion-libraries)
4. [Configuring the Job File](#4-configuring-the-job-file)
5. [Starting a HiPerGator Session](#5-starting-a-hipergator-session)
6. [Connecting via SSH Tunnel](#6-connecting-via-ssh-tunnel)
7. [Prompt Engineering for Crop Disease Images](#7-prompt-engineering-for-crop-disease-images)
8. [Running Stable Diffusion in Jupyter](#8-running-stable-diffusion-in-jupyter)
9. [Fine-Tuning Stable Diffusion on Crop Disease Data](#9-fine-tuning-stable-diffusion-on-crop-disease-data)
10. [Tips & Troubleshooting](#10-tips--troubleshooting)

---

## 1. Prerequisites

- Active UF account with access to [HiPerGator OnDemand](https://ood.rc.ufl.edu/pun/sys/dashboard/batch_connect/sessions)
- Duo Mobile installed on your phone
- Access to `/blue/$NAME$/YOUR_USERNAME/`
- Basic familiarity with terminal/command line
- A `.job` file configured (see Step 4)

---

## 2. Setting Up Your Conda Environment on HiPerGator

In this step, make sure you make a completely **new** python environment to install necessary packages pertaining to Stable Diffusion architecture training and generating images. If you use the old ones, the new packages may conflict and you may not be able to import the packages successfully!

### Steps

1. Go to [https://ood.rc.ufl.edu/pun/sys/dashboard/batch_connect/sessions](https://ood.rc.ufl.edu/pun/sys/dashboard/batch_connect/sessions)
2. Click **Clusters** → **HiPerGator Shell Access**
3. Load Conda in the terminal and run the lines below. Also, wait for all the packages to install. It may take some time and would ask to prompt 'Y' for Yes.

```bash
module load conda
mamba create -p /blue/$NAME$/$YOUR_USERNAME$/Environment/$CUSTOM_PACKAGE$/ python=3.11 # I usually keep all my packages within a folder called 'Environment' so its organized in once place and easy for me to import and run a job file when needed.
```
4. Once your custom name environment is created, ensure it is activated with the command below,

```bash
mamba activate /blue/$$NAME/$YOUR_USERNAME$/Environment
```

Once the environment is activated, you will see the line change below based on the path you gave to activate your environment,

```text
(/blue/$NAME$/$YOUR_USERNAME$/Environment/Stable_Diffusion3) [$YOUR_USERNAME$@login10 ~]$
```
