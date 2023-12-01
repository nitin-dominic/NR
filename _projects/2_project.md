---
layout: page
title: Stand count automation
description: A project that uses transfer learning to count sunflower stands in drone-acquired imagery.
img: assets/img/tl1.jpg
importance: 2
category: work
---

# Crop stand count automation using transfer learning in ArcGIS Pro

### Background

This algorithm imports Arcpy, and Rasterio packages in Python to create a weed prescription map. A feature to calculate Excess Green and 4 different VIs, namely, Normalised Difference Vegetation Index (NDVI), Normalized Difference Red Edge Index (NDRE), Soil Adjusted Vegetation Index (SAVI), and Optimized Soil Adjusted Vegetation Index (OSAVI) can also be calculated and exported to local drive using this script. The script accepts either a multispectral imagery or an RGB imagery. Based on band count, it calculates the required indices. After indices calculation, it also performs image sharpening and thresholding, thereby converting the whole image into a binary one. After that it converts objects within the image to polygons and perfroms weed identification while creating a weed map using Fishnet Grid technique. Full Python code to automate the process can be found [here](https://github.com/nitin-dominic/Automating-development-of-weed-prescription-map-using-Arcpy-and-Python/blob/main/WeedMappingScript.py).

### Prerequisites:

1. RGB or multispectral imagery should be provided by the user.
2. Specify a directory with an empty folder before this scripts starts exporting all the indices (output images) or processed images.
3. Shapefile (a polygon line drawn over the crop plants) should be provided by the user.

### Limitation of this algorithm:

1. Weeds won't get detected or identified if they are present in-between crop rows.

### Credits:

1. Original workflow in ArcGIS Pro was developed by Dr. J. Paulo Flores (Assistant Prof., NDSU)
2. Automation using Python scripting was developed by Nitin Rai

### Final out of the developed weed prescription map: 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/pm.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    In the displayed figure, green represents corn crops and red represent weeds.
</div>
