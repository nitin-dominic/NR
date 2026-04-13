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
5. [Starting a HiPerGator Session and Connecting via SSH Tunnel](#5-starting-a-hipergator-session-and-connecting-via-SSH-tunnel)
6. [Running Stable Diffusion in Jupyter](#8-running-stable-diffusion-in-jupyter)
7. [Fine-Tuning Stable Diffusion on Crop Disease Data](#9-fine-tuning-stable-diffusion-on-crop-disease-data)
8. [Tips & Troubleshooting](#10-tips--troubleshooting)

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

```console
module load conda
mamba create -p /blue/$NAME$/$YOUR_USERNAME$/Environment/$CUSTOM_PACKAGE$/ python=3.11 # I usually keep all my packages within a folder called 'Environment' so its organized in once place and easy for me to import and run a job file when needed.
```
4. Once your custom name environment is created, ensure it is activated with the command below,

```console
mamba activate /blue/$$NAME/$YOUR_USERNAME$/Environment
```

Once the environment is activated, you will see the shell prompt change below based on the path you gave to activate your environment,

```console
(/blue/$NAME$/$YOUR_USERNAME$/Environment/Stable_Diffusion3) [$YOUR_USERNAME$@login10 ~]$
```
Once you see this, go head and start installing all the required packages. At this point, I would also suggest to please consider visiting [Hugging Face diffusers doc](https://huggingface.co/docs/diffusers/index) and repo to gain more information on which packages to install and how to get started. I am anticipating that a few things pertaining to how you train and deploy Stable Diffusion model for image generation may have changed. Also, in my case on the HPC, CUDA was already installed and I did not bother myself changing any versions of it. I checked it via importing torch package. I would recommend doing the same. However, I am giving you a starting point to install packages and get started. See below!

```python
pip install torch torchaudio torchvision numpy scipy opencv matplotlib glob notebook diffusers transformers accelerate safetensors xformers huggingface_hub Pillow # installing notebook is important since you won't be able to run Jupyter Notebook!
```
---

## 4. Configuring the Job File

So, what is a job file? Consider a job file like placing an order at a restaurant. Likewise, HiPerGator is a shared supercomputer and so hundreds of researchers use it simultaneously. Rather than running your code directly on the machine, you submit a job file *(.job)* that tells the scheduling system (SLURM) exactly what resources you need: how many CPUs, how much RAM, which GPU, and for how long. Upload your .job file via HiPerGator Home Directory which will look like below. 

```bash
#!/bin/bash
#SBATCH --job-name=jupyter
#SBATCH --output=deep_learning_models.log # Your custom name of the log file using which you will add your credentials to upload your scripts
#SBATCH --cpus-per-task=10
#SBATCH --mem=64gb # RAM
#SBATCH --time=120:00:00 # time you would need to finish the job
#SBATCH --partition=hpg-b200
#SBATCH --gpus=1
date;hostname;pwd
module load conda
conda activate /path/to/the/environment/you/created/
port=$(shuf -i 20000-30000 -n 1)
echo -e "\nStarting Jupyter Notebook on port ${port} on the $(hostname) server."
echo -e "\nSSH tunnel command:"
echo -e "\tssh -NL ${port}:$(hostname):${port} ${USER}@hpg.rc.ufl.edu"
echo -e "\nLocal browser URI:"
echo -e "\thttp://localhost:${port}"
host=$(hostname)
jupyter-notebook --no-browser --port=${port} --ip="$host"
```
---

## 5. Starting a HiPerGator Session and Connecting via SSH Tunnel

Go to [https://ondemand.rc.ufl.edu/pun/sys/dashboard](https://ondemand.rc.ufl.edu/pun/sys/dashboard), and click on HiPerGator Shell Access thorugh Clusters tab. Once in, submit your job using the command line below.

```console
sbatch $YOUR_JOB_FILE_NAME$.job
```
Once done, you can check the status of your job file from Jobs -> Active Jobs. Alternatively, a log file will appear based on the filename you gave in the job file. Open that and copy paste the line (just the ssh line below) in your command prompt. Once propted, go ahead and put your username and password and login. 

```console
SSH tunnel command:
	ssh -NL 20105:c1100a-s15.ufhpc:20105 nitin.rai@hpg.rc.ufl.edu
```
After login is successful, open thje log file again, and copy paste the jupyter notebook line (below) and start your notebook.

```console
http://127.0.0.1:20105/tree?token=39536641ca222572eb10e1a668dab7b133e55bc97bcd08db # this could be different on your machine for the IP address.
```
---

## Running Stable Diffusion in Jupyter Notebook

Below is the snippet you will find on [Stable Diffusion HiperGator](https://github.com/nitin-dominic/Stable_Diffusion_HiPerGator). I will go briefly with a specific focus on prompt engineering. 

```python
!accelerate launch train_dreambooth_lora_sdxl.py \
  --pretrained_model_name_or_path="stabilityai/stable-diffusion-xl-base-1.0" \ #You can change this to SD3.5M or L based on the correct argument. 
  --pretrained_vae_model_name_or_path="madebyollin/sdxl-vae-fp16-fix" \ #keep it as it is when fine-tuning SDXL. 
  --instance_data_dir="/path/to/the/training/data" \
  --output_dir="/path/to/output/directory/" \
  --instance_prompt="A dense canopy of field-grown watermelon vines with leaves showing realistic anthracnose (Colletotrichum orbiculare) lesions. Irregular brown, sunken spots along veins and margins, captured with a handheld Canon DSLR in bright natural sunlight. Fine leaf texture, hairy stems, natural shadows, light field background board, and diverse leaf angles resembling real ground-level field photography" \
  --resolution=1024 \
```
