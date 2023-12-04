---
layout: page
title: Project 3
description: DarkNet Framework-based Weed Detection for Edge Device Implementation
img: assets/img/ragweed4.png
redirect: 
importance: 1
category: work
---

# DarkNet Framework-based Weed Detection for Edge Device Implementation

This repository consists of open-source DarkNet-based implementation of the paper published in the Journal of the ASABE. You can read the paper [here](https://elibrary.asabe.org/abstract.asp?AID=54375&t=3&dabs=Y&redir=&redirType=). It consists of the configuration files and the dataset in YOLO (*.txt*) format for other researchers to develop and deploy Deep Learning-based weed detection approach. Please feel free to use the dataset but do remember to cite us in your work!

The configuration files (lightweight architecture) used in this research study needs to be merged with the original repository of [DarkNet YOLOv4](https://github.com/AlexeyAB/darknet). Once you have cloned the repository, you can bring in the dataset and start training the dataset with appropriate hyperparameter tunings/tweakings. We have also added a folder called Validation_metrics which displays all the screenshots of the metrics obtained after training on our dataset. Please refer to the original repository for more details.  

The dataset used in the paper can be found at Mendeley Data (open-source dataset repository). Please download the files [here](https://data.mendeley.com/datasets/8kjcztbjz2/2).

### Sample of weed image dataset used in this ressearch study and published on Mendeley Data:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/ragweed11.png" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/ragweed4.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Represents high-resolution aerial images used for annotation purpose. Following weeds are represented by the colors, (a) ragweed (yellow), (b) kochia (red), and (c) horseweed (violet).
</div>

### Reference
**Rai, N.,** Sun, X., Igathinathane, C., Howatt, K., Ostlie, M. (2023). Aerial-based weed detection using low-cost and lightweight deep learning models on an edge platform. ***J. ASABE***., ***66***(5). DoI: [https://doi.org/10.13031/ja.15413](https://doi.org/10.13031/ja.15413).
