Colab notebook links: [Detectron2](https://colab.research.google.com/drive/1w7zePGRXP7RHT7znzh964aeLGKecaXWp?usp=sharing), [YOLOv8](https://colab.research.google.com/drive/1K-MpS1x4mS4RA4qr_X3ASTG_g1mFB29c?usp=sharing).

# 1. Dataset

I collected a dataset of 38 photos with books (190 instances) and cars (94 instances). 

Examples of annotated photos:

<img src="https://user-images.githubusercontent.com/48735488/234954046-5b9d41d3-c856-4791-bcf7-823237c29611.png" width=35% height=35%>
<img src="https://user-images.githubusercontent.com/48735488/234954207-df96f3c4-8b47-4a11-8f07-ac9198e81d81.png" width=35% height=35%>

# 2. Dataset Annotation and Augmentations

I annotated the dataset on Roboflow.

At the dataset generation step, I applied the following preprocessing:
* Auto-orientation of pixel data (with EXIF-orientation stripping)
* Resize to 640x640 (Stretch)

And the following augmentations:

* Randomly crop between 0 and 25 percent of the image
* Random rotation of between -10 and +10 degrees
* Random brigthness adjustment of between -20 and +20 percent
* Random exposure adjustment of between -15 and +15 percent
* Random Gaussian blur of between 0 and 1 pixels

Examples of annotated augmented samples:

<img src="https://user-images.githubusercontent.com/48735488/234953743-abb9a84c-a4e7-4f35-8293-5c4c8adb1972.png" width=35% height=35%>
<img src="https://user-images.githubusercontent.com/48735488/234953828-937a84ac-192e-4f8e-94e8-1ed7acdddddb.png" width=35% height=35%>


# 3. Mask R-CNN

I trained a Mask R-CNN model from Detectron2. In particular, its `mask_rcnn_R_50_FPN_3` version.

I ran 10000 iterations, however the training process was early stopped after 2679 iters (no loss decrease for 500 iters). The results are described at the point 5.

# 4. YOLOv8

I also trained a YOLOv8 model for segmentation. I chose the smallest size architecture `yolov8n-seg.pt`. It has 3,2 million parameters.

I ran 300 epochs. However, the early stopping was triggered after 203 epochs (no improvements during last 50 epochs). The best model was obtained after the epoch 153.

# 5. Results

Both models were evaluated based on mAP, speed of inference, and size.

## Mask R-CNN

- **mAP:** 0.32898

- **mAP50:** 0.44665

- **Speed of inference:** 146.42 ms

- **Model Size:** 170 MB

Evaluation results (averaged between classes):
|   AP   |  AP50  |  AP75  |  APs  |  APm   |  APl   |
|:------:|:------:|:------:|:-----:|:------:|:------:|
| 32.898 | 44.665 | 39.985 | 0.000 | 30.848 | 41.073 |

Evaluation results (per-class):
| category   | AP     | category   | AP    |
|:-----------|:-------|:-----------|:------|
| book       | 65.797 | car        | 0.000 |

As you can see, there is no progress with cars segmentation.

Possible reasons are:

1. Class imbalance:

|  category  | #instances   |  category  | #instances   |
|:----------:|:-------------|:----------:|:-------------|
| book       | 470          |    car     | 282          |
|   total    | 752          |            |              |

2. Poor Dataset Quality:

In the original dataset, there are 94 bboxes with cars (which were augmented to 282 samples, as you see in the table above). Among these 94 examples, there are plenty of ways a car was captured at the photo: side, back, front, diagonally, from afar, from near. Such variability together with lack of data could lead to poor performance within the limited number of epochs.

The books case is much easier: all of them were captured from approximately same distance and angle. Moreover, there are more books samples in the dataset. So, the model has managed to converge well during the training process. The resulted `mAP` for the books class is 0.66.


## YOLOv8

- **mAP50-95:** 0.489

- **mAP50:** 0.839

- **Speed of inference:** 12.6 ms (27.8 ms with post-processing)

- **Model Size:** 6.5 MB

Details:

|Class     |Images    |Instances   | Precision  | Recall    |   mAP50    | mAP50-95 |
|:--------:|:---------|:----------:|:-----------|:---------:|:-----------|:--------:|
|all       |   6      |     36     |  0.853     |  0.812    |   0.839    |  0.489   |
|book      |   6      |     16     |  0.831     |  0.924    |   0.931    |   0.57   |
|car       |   6      |     20     |  0.874     |  0.7      |   0.747    |  0.409   |
