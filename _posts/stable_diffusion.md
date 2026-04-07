# HiPerGator Environment Setup & Deep Learning Tutorial

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Creating a New Conda Environment](#1-creating-a-new-conda-environment)
3. [Installing Libraries](#2-installing-libraries)
4. [Setting Up a Jupyter Notebook Kernel](#3-setting-up-a-jupyter-notebook-kernel)
5. [Configuring the Job File](#4-configuring-the-job-file)
6. [Starting a HiPerGator Session](#5-starting-a-hipergator-session)
7. [Connecting via SSH Tunnel](#6-connecting-via-ssh-tunnel)
8. [Fine-Tuning Your Model](#7-fine-tuning-your-model)
9. [Tips & Troubleshooting](#8-tips--troubleshooting)

---

## Prerequisites

- Active UF account with access to [HiPerGator OnDemand](https://ood.rc.ufl.edu/pun/sys/dashboard/batch_connect/sessions)
- Duo Mobile installed on your phone
- Access to `/blue/nsboyd/YOUR_USERNAME/`
- Your `.job` file configured (see Step 4)
- Basic familiarity with terminal/command line

---

## 1. Creating a New Conda Environment

> **Why?** Different models (YOLOv5, Mask-RCNN, Faster-RCNN, DeepLabV3+, etc.) require different Python packages. Separate environments prevent version conflicts.

### Steps:

1. Go to: [https://ood.rc.ufl.edu/pun/sys/dashboard/batch_connect/sessions](https://ood.rc.ufl.edu/pun/sys/dashboard/batch_connect/sessions)
2. Click **Clusters** → **HiPerGator Shell Access**
3. In the terminal, load Conda:

```bash
module load conda
