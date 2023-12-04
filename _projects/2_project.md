---
layout: page
title: Project 2
description: A project that uses transfer learning technique to count sunflower stands in drone-acquired imagery
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
2. Data loading from the directory
```
RGB_imagery = r"E:\Nitin.Rai\ASABE2021\Dataset\sunflower_Clip_2.tif"
arcpy.env.workspace = RGB_imagery
bands = [Raster(os.path.join(RGB_imagery, b))
         for b in arcpy.ListRasters()]
```
3. Preparing the dataset for training task

The provided Python code snippet appears to be setting up data preparation for a computer vision task involving image chips. The below code snippet uses [KITTI](https://www.cvlibs.net/datasets/kitti/) rectangles dealing with the rectanglular shapes of teh identified objects in the imgery. Caution: Set all these parameters based on your understanding and imagery requirements.

```
dataset_path = r"C:\Users\Nitin.Rai\Desktop\ImageChipsPASCAL"
class_mapping = None
chip_size = 448
val_split = 0.3
batch_size = 64
transform = False
dataset_type = 'KITTI_rectangles'
prepare = prepare_data(dataset_path, class_mapping, chip_size, val_split, batch_size, transform, dataset_type)
```

4. Exporting the image chips for training
The provided Python code snippet is using the ExportTrainingDataForDeepLearning tool from the ArcGIS library to prepare training data for deep learning models. This tool is commonly used for creating training datasets for object detection tasks.

```
raster_dataset = "D:/ArcGIS Projects/Dataset/SunflowerCropImagery.tif"
output = "D:/ArcGIS Projects/Dataset/OutputFolder2233445566"
training_samples = "D:/ArcGIS Projects/Dataset/CropClassification.shp"
chip_format = "TIFF"
tile_sizeX = "448"
tile_sizeY = "448"
stride_sizeX = "224"
stride_sizeY = "224"
nofeature_tiles = "ONLY_TILES_WITH_FEATURES"
metadeta_format = "PASCAL_VOC_rectangles"
start_index = 0
classvalue_field = "Classvalue"
buffer_radius = 0
input_mask_polygons = ""
rotation_angle = 10
reference_system = "MAP_SPACE"
processing = "PROCESS_ITEMS_SEPARATELY"
blacken = "NO_BLACKEN"
crop_mode = "FIXED_SIZE"

ExportTrainingDataForDeepLearning(raster_dataset, output, training_samples,
                                  chip_format, tile_sizeX, tile_sizeY, stride_sizeX,
                                  stride_sizeY, nofeature_tiles, metadeta_format, start_index, 
                                  classvalue_field, buffer_radius, 
                                  input_mask_polygons, rotation_angle, reference_system,
                                  processing, blacken, crop_mode)
```


5. Setting up hyperparameters to train the model(s):
The provided Python code snippet is training a deep learning model for object detection using RetinaNet. You can chose from various [pre-trained models](https://www.esri.com/en-us/arcgis/deep-learning-models) available within ArcGIS Pro environment. 

```
start_time = datetime.now()
dataset = r'E:\Nitin.Rai\ASABE2021\Dataset\ImageChips'
output_location = r'E:\Nitin.Rai\SunflowerCropDetection\ModelOutput2\RetinaNetReTrained'
eph = 50
objectDetection_model = "RETINANET"
batch_size = 64
arg = "SCALES '[1,1,0.8]'; RATIOS '[0.5,1,1.5]'; chip_size:416"
lr = 0.001
backbone_model = "RESNET50"
pretrained_model = None
validate = 20
stop_training = "STOP_TRAINING"
freeze = "UNFREEZE_MODEL"
TrainDeepLearningModel(dataset, output_location, eph, objectDetection_model, batch_size, arg, lr, backbone_model,
                       pretrained_model, validate, stop_training, freeze)
end_time = datetime.now()
print("Duration of Training the dataset was: {}" .format(end_time - start_time))
```

### Final ouput of the aerial imagery 

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
