---
layout: post
title: Object Detection in UAS-acquired Imagery Using ArcGIS Pro and Deep Learning
date: 2020-08-20 09:56:00-0400
description: 
tags:
categories:
---

This write up/tutorial is for those who are currently involved with working on ArcGIS Pro and want to learn a bit about Deep Learning too. Although, Deep Learning can be executed and worked independently using Python and other common platforms, I’ll explain how can we integrate Deep Learning in ArcGIS Pro. If you already know how to do that, you may even choose to skip reading the write up. ArcGIS Pro has recently released 2.6 version which involves installing different newer version of Deep Learning packages within ArcGIS Pro. What needs to be noted down here is that there are several specific package versions of Deep Learning tools for ArcGIS Pro 2.5v and 2.6v. Pay attention while installing those packages because even if you miss out one package version you will end up in a lot of errors which is probably not desired to make you feel more frustrated. I have jotted down all the specific version for ArcGIS Pro 2.5v and 2.6v. Not only this but also, I have included few codes which you can write in python (just to automatize and save some time without much clicks!). Although you will find all these instructions on ESRI website (Deep Learning in ArcGIS Pro), you may have to browse through a lot of web pages back and forth to gather information from all sides. I have included all the details right here needed to integrate Deep Learning in ArcGIS Pro. 

{% capture notice-text %}Before starting, please be mindful that the deep learning packages installed in this blog are outdated. Please check the latest versions within the software and install accordingly!{% endcapture %}
<div class="notice--danger">
{{notice-text | markdownify}}
</div>

### 1. Installing Deep Learning Tools in ArcGIS Pro

**a. ArcGIS Pro (2.5v)**

1. To begin, download Anaconda with a Python 3.6v (as I did in my case)

2. Open Python Command Prompt and write these lines *(italicized)*

In the place of deeplearning_arcgispro you can put any name you want. This creates an environment and clones everything from arcgispro-py3 which is already present in ArcGIS Pro folder when you initially installed it. This will also take few minutes to clone. After you have successfully cloned arcgispro-py3, you can see it by following this path, C:\Users\<username>\AppData\Local\ESRI\conda\envs\deeplearning. Also please install all these in a newly created environment (folder).

```conda create –name deeplearning_arcgispro –clone arcgispro-py3```

now activate the created deeplearning_arcgispro envs

```activate deeplearning_arcgispro```

begin installing the packages (be specific with the versions here). Also, for those who doesn’t own a PC with Nvidia GPU and wish to run TensorFlow on a CPU instead of a GPU, you can add a package called `“tensorflow-mkl”` from the Python Package Manager in ArcGIS Pro itself.

```conda install tensorflow-gpu=1.14.0```

```conda install keras-gpu = 2.2.4```

```conda install scikit-image=0.15.0```

```conda install Pillow = 6.1.0```

```conda install fastai = 1.0.54```

```conda install pytorch=1.1.0```

```conda install libtiff=4.0.10 –no-deps```

```proswap deeplearning_arcgispro```

Next time you’ll run ArcGIS Pro, click on Python in the opening window and click on Manage Environments. You’ll notice that the software has switched its active environment to your created environment, i.e., `deeplearning_arcgispro`

**b. ArcGIS Pro (2.6v)**

Everything remains the same except the package versions. Follow everything except a few changes when typing the commands, so instead use,

conda install tensorflow-gpu=2.1.0

conda install keras-gpu = 2.3.1

conda install scikit-image=0.17.2

conda install pillow-simd=7.0.0

conda install fastai=1.0.60

conda install pytorch=1.4.0

conda install libtiff=4.0.10

conda install torchvision=0.5.0

### 2. Creating labels and exporting data for Deep Learning

This is the hardest and most time-consuming part of using Deep Learning in ArcGIS Pro. But if done sincerely and with patience can yield a good model. Now you’re going to manually create datasets for training and validation purpose. Always remember, the higher the datasets the better the model predicts or detects objects of interest.

Begin with adding an imagery in ArcGIS Pro. Add an RGB imagery (can be a multispectral imagery with NIR & RedEdge Bands too but I haven’t worked on it yet). After you have successfully added the imagery,

1. Click on Imagery tab and click on Classification Tools and finally click on Label Objects for Deep Learning.

2. Once you click it, a new side window opens with Image Classification Specifications and new schema. Right click on new schema and click edit properties. Under edit properties add a class name (usually what you want the machine to detect for you). Once done, save it! You’ll see that the newly created Schema shows up on the screen within the side bar. Right click on that named schema and “Add a class”. This is basically creating images for different class types. Give it a name of the object you want to detect, give a value (usually 1) and color of your choice. Click on OK.

3. Now you’ll see different set of tools above your created class, click on one of those according to your choice. It can be even hand-free for object delineation.

4. After this step, edit objects (by hand) which you want your model to detect it for you. Carefully try to collect as much data as possible. Again, the datasets should be huge to build a good model.

5. Within the Image Classification side bar, you’ll see the classes being created along with the pixel percent. After you have finished editing the objects, click on save (middle purple floppy) button. Under projects, click folders, click whatever name you have used to save the project and inside this give a feature class name.

6. Once that is done, click on Export Training Data beside Labeled Objects in the same Image Classification sidebar,

Output Folder: Browse to the same Projects/Folders/<Name of your project>/ImageChips (create this folder).

Image Format: JPEG (if you’re writing a code in Python, this is what the file type that the code will accept. I remember giving .tiff once and it threw an error stating that the parameters are not valid).

Tile Size X: 256

Tile Size Y: 256

Stride X: 128

Stride Y: 128

Rotation Angle: 0 (you can change if you want)

Meta Data Format: PASCAL Visual Object Classes (specifically for object detection)

7. Run it! If you get an error here, there are probably 3 reasons,

a. Either the versions of packages been installed are not appropriate, and the environment created, (this one is very very common issue),

b. Problem with Output Folder specification (always use a newly made folder), or,

c. Image Format.

Alternatively use command line interface in Jupyter to Export your data

https://pro.arcgis.com/en/pro-app/tool-reference/image-analyst/export-training-data-for-deelearning.htm

### 3. Training the exported data to build a model

Now, ArcGIS Pro exports several files along with Images of your object of interest under ImageChips folder you made before. One of the files most important for performing Deep Learning is the .emd (ESRI Model Definition) file. This file is a passage that connects ArcGIS Pro and Deep Learning. You can even choose to edit this file and use TensorFlow, Keras according to you need and work. I’m planning in my next blog to write about how to edit these files and perform deep learning. Once you have the folder with you, you can choose to train your model either in the ArcGIS Pro Geoprocessing Tool (by typing Train Deep Learning Model) or Python. I did it in Python just to learn and visualize the interface during learning and prediction time. Below is my attached screenshot while training the data in Jupyter.


Fig.2. Training data for object detection in ArcGIS Pro (v 2.5)
If you’re using Geoprocessing tab (by clicking on Train Deep Learning Model tool, Image Analyst) in ArcGIS Pro to build a model, you can populate the required fields as follows,

Input Training Data — You’ll add the ImageChips folder here which contains the images and .emd file as I described above

Output Model — Make an empty folder and name it as per your choice

Max Epochs — Default is 20 but I would recommend if you need a good accuracy go for a higher number, let’s say, 100. This has a direct connection with your GPU type you’re choosing. If it’s a powerful GPU, it won’t take much time.

Model Parameters:

Model Type: SSD (or RETINET for object detection). Don’t choose any other types as not all the models present are used for object detection.

Batch Size: 2 (or maybe even 8, 16, 32 based on the system you’re using)

If using SSD, specify grids [4, 2, 1], zooms [0.7, 1, 1.3] and ratios [[1, 1], [1, 0.5], [0.5, 1]] as default specifications.

Advanced:

Backbone Model — ResNet 34 (or ResNet 50)

Leave Pre-trained model as of now if you’re doing it for the first time.

Validation — Can be 10 or 20

Leave everything as same and Run!

Note: Now if you’re again getting an error, it is just because of those 3 reasons which I discussed earlier in this file. Try implementing it again. If you get all of this in one go, you’ll be happy. But if not, it’s going to make you feel a lot frustrated.

### 4. Detecting objects using the trained model

Once everything is done successfully, all you have to do is to open ArcGIS pro again and go to Analysis -> Tools -> Detect Objects Using Deep Learning. You can even implement a code (as I did) just to click run and let the algorithm export a file for you with detected objects and a shape file. In ArcGIS pro, you’ll see these information as you click on Detect Objects Using Deep Learning,

Input Raster: Add your imagery here.

Output Detected Objects: A new folder specifying where you save the shape file for the detected objects. This is really useful!

Model Definition: Load your trained .emd file here.

Click on Non-Maximum Suppression: This boils down a lot of detected rectangles (overlapping) to a few.
