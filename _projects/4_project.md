---
layout: page
title: Project 5
description: Automating crop stand count using spatial tools in ArcGIS Pro
img: assets/img/cropstand.png
importance: 1
category: work
---

# Automating Crop Stand Count using Arcpy and Python
<b>Rai, N</b> & Flores, P

> ##### Tip:
> This project/script shares its concept from Project 1 except for the fact that it does not create weed prescription map.
{: .block-tip }

This algorithm imports Arcpy, and Rasterio packages in Python to count crops in each row in a given raster data. A feature to calculate Excess Green and 4 different VIs, namely, Normalised Difference Vegetation Index (NDVI), Normalized Difference Red Edge Index (NDRE), Soil Adjusted Vegetation Index (SAVI), and Optimized Soil Adjusted Vegetation Index (OSAVI) can also be calculated and exported to local drive using this script. The script accepts either a multispectral imagery or an RGB imagery. Based on band count, it calculates the required indices. After indices calcualtion, it also performs image sharpening and thresholding thereby converting the whole image into a binary image. After that it converts objects within the image to polygons and perfroms crop stand counts in each row. A .csv file containing number of crops (with unique IDs for each crops) in each rows can also be exported using this script. 

# Prerequisites: 
1. RGB or a multispectral imagery should be provided by the user.
2. Specify an directory with an empty folder before this scripts starts exporting all the indices (output images) or .csv files.
3. Shapefile (a polygon line drawn over the crops) should be provided by the user.

# Limitation of this algorithm: 
1. Weeds gets counted as crops if they are present in-between crops rows.

# Credits:
1. Original workflow in ArcGIS Pro was developed by Dr. J. Paulo Flores (Assistant Prof. at NDSU)
2. Automation using Python scripting was developed by Nitin Rai (PhD Student)

# Credits
1. Original concepts were formulated by Dr. J. Paulo Flores (Asst. Prof., NDSU)
2. Codes were developed by Nitin Rai
