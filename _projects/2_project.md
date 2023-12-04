---
layout: page
title: Project 2
description: A project that uses transfer learning to count sunflower stands in drone-acquired imagery
img: assets/img/tl1.jpg
importance: 1
category: work
---

# Crop stand count automation using transfer learning in ArcGIS Pro

### Background

This project was done using various ArcPy packages available within the ArcGIS Pro environment. It was mainly scripted to automate [transfer learning](https://pro.arcgis.com/en/pro-app/latest/tool-reference/image-analyst/train-deep-learning-model.htm) workflow in ArcGIS Pro for sunflower stand count automation. This script has been made open-source and is the backbone of the paper published [here](https://elibrary.asabe.org/abstract.asp?aid=52515). 

### Prerequisites:

1. RGB orthomosaic imgery.
2. Specify a directory with an empty folder before this scripts starts exporting all the indices (output images) or processed images.
3. Properly installed deep learning packages in ArcGIS Pro environment. Check my [blog](https://nitin-dominic.github.io/NR/blog/2020/ArcGISPro/) for further clarification.

### Limitation of this algorithm:

1. Demands a strong GPU with a good memory (12 GB recommended!)
2. Algorithm has not been tested on multispectral (MS) aerial imgery. We are not sure of the models could be deployed to locate and count sunflower stand in MS imagery. 

### Code breakdown
1. Importing all the required packages
```
import arcpy,
from arcpy.ia import *,
    "import os, sys,
    "from arcgis.gis import GIS,
    "from arcgis.learn import export_training_data,
    "from arcgis.learn import prepare_data,
    "from arcgis.learn import YOLOv3,
    "from arcgis.learn import FasterRCNN,
```

### Final ouput of the developed weed prescription map: 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/tl3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A screenshot of the aerial imagery after the trained and developed model was deployed for locating and counting purpose. Green represents single stands while red are doubles.
</div>

### Credits:

1. Imagery was given by Dr. J. Paulo Flores (Assst. Prof at NDSU)
2. Data processing pipeline, workflow, and automation using Python scripting was developed by Nitin Rai
