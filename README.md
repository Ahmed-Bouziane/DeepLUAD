# DeepLUAD
Diagnosis of histologic growth patterns of Lung Adenocarcinoma cancer in digital slides using deep learning.
![Solid visualization](/Images/Solid_visualization.png)

# Table of contents

* [General Information](#General-Information)
* [Requirements](#Requirements)
* [Installation](#Installation)
* [Usage](#Usage)
* [Known Issues and Limitations](#Known-Issues-and-Limitations)
* [Future Work](#Future-Work)
* [Sources](#Sources)

# General Information

- _This is a final year graduation project._

__1-Growth patterns classification :__

- We are using 26 whole-slide images obtained from [The Cancer Genome Atlas (LUAD)](https://portal.gdc.cancer.gov/projects/TCGA-LUAD).
- You can download the images using the [manifest file](/Extra/gdc_manifest_Lung_data.txt).
- Download annotations from [here](https://static-content.springer.com/esm/art%3A10.1038%2Fs41598-018-37638-9/MediaObjects/41598_2018_37638_MOESM2_ESM.zip).
- Annotations were acquired from the second [source](#Sources).
	* Distribution of data among histologic patterns is as follows:

__Histologic pattern__ | __ACINAR__| __CRIBRIFORM__ | __MACROPAPILLARY__ | __NON CANCEROUS__ | __SOLID__
-------------------|--------|------------|----------------|---------------|-----------------------------
__Crops__ | 22 | 4 | 23 | 53 |85
__Patches __ | 4507 | 921 | 5706  | 6222  | 5207

- We augmented data to 7000 patches per class.

__3-Lung cancer classification :__ 

- We used the same dataset __(LC25000)__ (https://arxiv.org/abs/1912.12142v1) dataset which contains 25000 patches of sizes (768*768*3), but we only worked with the lung cancer image sets.
- We did not perform any data augmentation.
 
# Requirements

- [OpenSlide Python](https://openslide.org/api/python/)
- [PIL](https://pillow.readthedocs.io/en/5.3.x/)
- [Python 3.6](https://www.python.org/downloads/release/python-360/)
- [PyTorch](https://pytorch.org/)
- [pytorch-gradcam](https://pypi.org/project/pytorch-gradcam/)
- [scikit-image](https://scikit-image.org/)
- [scikit-learn](https://scikit-learn.org/stable/install.html)
- [seaborn](https://seaborn.pydata.org/installing.html)
- [tensorboard](https://www.tensorflow.org/tensorboard)
- [torchvision](https://pytorch.org/docs/stable/torchvision/index.html#module-torchvision)

# Installation 

## Install packages:


```
sudo pip3 install -r requirements.txt
```


## Download repository:

```
git clone https://github.com/Ala-Eddine-BOUDEMIA/Lung-Cancer-Diagnosis.git
```


```
cd Lung-Cancer-Diagnosis/2-PFE_Modification/
```

## Folders structures:

    	Codes
    	│
	├────── 1-Growth patterns classification
	│	│
	│	├────── All_WSI 				
	│	│	├──── Folder_1/Image_1.svs
	│	│	├──── Folder_2/Image_2.svs
	│	│	└──── Folder_n/Image_n.svs
	│	│
	│	├────── Annotations
	│	│	├──── Annotation_Image_1.xml
	│	│	├──── Annotation_Image_2.xml
	│	│	└──── Annotation_Image_n.xml
	│	│
	│	├────── CSV_files
	│	│	├──── Annotations
	│	│	├──── Diagnostics
	│	│	└──── Predictions
	│	│
	│	├────── Patches
	│	│	├──── ACINAR
	│	│	├──── CRIB
	│	│	├──── MICROPAP
	│	│	├──── NC
	│	│	└──── SOLID
	│	│
	│	├────── Train_folder
	│	│	├────── Model
	│	│	│	├──── Best_model_weights
	│	│	│	└──── Checkpoints
	│	│	│
	│	│	├────── Test_patches
	│	│	├────── Train_patches
	│	│	└────── Validation_patches
	│	│
	│	├────── Tensorboard
	│	│
	│	└────── Visualization
	│		└────── Patchs
	│			├────── folder_1
	│			│	└──── Image_1_visualization.tiff
	│			├────── folder_2
	│			│	└──── Image_2_visualization.tiff
	│			└────── folder_n
	│				└──── Image_n_visualization.tiff
	│
	│	
	└────── 2-Lung cancer classification
		│
		├────── CSV_files
		│	├──── Annotations
		│	├──── Diagnostics
		│	└──── Predictions
		│
		├────── lung_colon_image_set 				
		│	└────── lung_image_sets
		│		├──── lung_aca
		│		├──── lung_n	
		│		└──── lung_scc
		│
		├────── Patches
		│	├──── lung_aca
		│	├──── lung_n
		│	└──── lung_scc
		│
		├────── Train_folder
		│	├────── Model
		│	│	├──── Best_model_weights
		│	│	└──── Checkpoints
		│	│
		│	├────── Test_patches
		│	├────── Train_patches
		│	└────── Validation_patches
		│
		├────── Tensorboard
		│
		└────── Visualization
			└────── Patchs
				├────── folder_1
				│	└──── Image_1_visualization.jpeg
				├────── folder_2
				│	└──── Image_2_visualization.jpeg
				└──── folder_n
					└──── Image_n_visualization.jpeg

# Usage

- Take a look at `Config.py` before you begin to get a feel for what parameters can be changed.

## 1. Preprocessing:

This code from __1-Growth patterns classification__ is meant to :
- Read from `Annotations` folder that contains XML annotation files.
	- XML files must be directly contained in the `Annotations` folder.
	- For example : `Annotations/annotation_1.xml`
- Read from `All_WSI` folder that contains Whole Slide Images.
	- Make sure that the WSIs are contained in at least one subfolder in the `All_WSI` folder.
	- For example : `All_WSI/WSI_1/Image.svs`
- Generates patches and saves information about the patches in a csv file.


```
python3 1-Preprocessing.py
```


**Inputs**: `All_WSI`, `Annotations`

**Outputs**: `Patches/SUBTYPE`, `CSV_files/Annotations`

- Note that: `SUBTYPE == ACINAR, CRIB, MICROPAP, NC, SOLID`.

If your histopathology images are H&E-stained, whitespace will automatically be filtered. 
You can change overlapping area using the `--Overlap` option.

## 1. Resize:

This code from __2-Lung cancer classification__ is meant to :
- Parse the image sets in `lung_image_set` folder.
- Resize images from (768*768*3) to (224*224*3).
- Save the resized images in `Patches/subtype` 


```
python3 1-Resize.py
```


**Inputs**: `lung_image_set`

**Outputs**: `Patches/subtype`

- Note that: `subtype == lung_aca, lung_n, lung_scc`.

## 2. Processing:

The goal of this code is to balance data using data augmentation techniques.

- Reads patches randomly from each subtype directory at a time.
- Applies diffrent transformations to the image.
- The nature and number of transformations applied to an image are chosen randomly.
- Modified patches are saved in the same directory as the original image.


```
python3 2-Processing.py
```


- Note that this may take some time and eventually a significant amount of space. 
	- Change `--Maximum` to be smaller if you wish not to generate as many windows. 
	- Make sure that `--Maximum` is not more than __15 times__ greater than the initial number of patches contained in the least represented class. 

**Inputs**: `Patches/SUBTYPE`

**Outputs**: `Patches/SUBTYPE`

## 3. Split:

Splits the data into a train, validation and test set. Default validation and test patches per class is 1000.
You can change these numbers by changing the `--Validation_Set_Size` and `--Test_Set_Size`. 
You can skip this step if you did a custom split.

Note that the modified images will be ditributed to the same set as the original, so the model won't be memorizing patterns.


```
python3 3-split.py
```


**Inputs**: `Patches` 

**Outputs**: `Train_folder/Train_patches`, `Train_folder/Validation_patches`, `Train_folder/Test_patches`

## 4. Train_val:

We recommend using ResNet-18 if you are training on a relatively small histopathology dataset. You can change hyperparameters using the `argparse` flags. There is an option to retrain from a previous checkpoint. Model checkpoints are saved by default every epoch in `Train_folder/Model/Checkpoints`.


```
python3 4-Train_val.py
```


**Inputs**: `Train_folder/Train_patches`, `Train_folder/Validation_patches`

**Outputs**: `Train_folder/Model/Checkpoints`, `Train_folder/Model/Best_model_weights`, `CSV_files/Diagnostics`, 
`Tensorboard`

## 5. Test:

Run the model on all the patches for each WSI in the test set.


```
python3 5-test.py
```


We automatically choose the model with the best validation accuracy while training. You can also specify your own. 

**Inputs**: `Train_folder/Test_patches`

**Outputs**: `CSV_files/Diagnostics`, `CSV_files/Predictions`, `Tensorboard`

## Tensorboard

We are using tensorboard to evaluate the model. 
- Uploads a grid of train and validation images to make sure that the patches are good. 
- Uploads the model's graph.
- Uploads confusion matrix and classification report of each epoch.
- Plots the loss function.
- Plots precision recall curve for each class. 

to run Tensorboard write the following in your Terminal and it will create a local host for you.


```
tensorboard --logdir=Tensorboard
```

## 6. Evaluation:

Aggregates the patches predictions from the Test code to predict a label at the whole-slide level.
There are various methods to do so, we decided to perform patch averages. Therefore we average the probabilities of all patch predictions, and take the class with the highest probability.


```
python3 Code/6-Evaluation.py
```


**Inputs**: `CSV_files/Predictions`

**Outputs**: `CSV_files/Predictions_cleaned`, `CSV_files/WSI_Name_Prediction.csv`

Note that `WSI_Name_Prediction` refers to actaul name of the WSI in question.

## 7. Visualization

This code allows to see what the network is looking at is to visualize the predictions for each class.

Note that The visualization is a patch level visualization using GradCAM.


```
python3 Code/7-visualization.py
```


**Inputs**: `CSV_files`, `Train_folder`

**Outputs**: `Visualization/Patchs`

# Known Issues and Limitations

- `3_Split` Takes a lot of time since it is a naive implementation.
- `4_Train_val` should have a better way to save the best model weights.

# Future Work

- [ ] Try diffrent architectures.
- [ ] Optimize the code to :
	* To reduce computation time.
	* To support multiprocessing.
	* To handle diffrent situations.
- [ ] Visualize on WSI level.
- [ ] Create a web interface. 

# License
[GNU General Public License v3.0](https://choosealicense.com/licenses/gpl-3.0/)
