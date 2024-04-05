# K-Radar Dataset
This is the documentation for how to use our detection frameworks with K-Radar dataset.
We tested the K-Radar detection frameworks on the following environment:
* Python 3.8.13 (3.10+ does not support open3d.)
* Ubuntu 18.04/20.04
* Torch 1.11.0+cu113
* CUDA 11.3
* opencv 4.2.0.32
* open3d 0.15.2

## Preparing the Dataset

* Via our server

1. To download the dataset, log in to <a href="http://QuickConnect.to/kaistavelab"> our server </a> with the following credentials: 
      ID       : kradards
      Password : Kradar2022
2. Go to the "File Station" folder, and download the dataset by right-click --> download.
   Note for Ubuntu user, there might be some error when unzipping the files. Please check the "readme_to_unzip_file_in_linux_system.txt".
3. After all files are downloaded, please arrange the workspace directory with the following structure:
```
KRadarFrameworks
      ├── tools
      ├── configs
      ├── datasets
      ├── models
      ├── pipelines
      ├── resources
      ├── uis
      ├── utils
      ├── logs
├── kradar_dataset (We handle the K-Radar dataset with multiple HDDs due to its large capacity.)
      ├── dir_2to5
            ├── 2
            ...
            ├── 5
      ├── dir_1to20
            ├── 1
            ├── 6
            ...
            ├── 20
      ├── dir_21to37
            ├── 21
            ...
            ├── 37
      ├── dir_38to58
            ├── 38
            ...
            ├── 58
```

* Via Google Drive Urls

The `K-Radar` dataset is being uploaded to Google Drive. It will take time to upload our entire dataset.
The 4D Radar tensor is only available on our server because it is too large to upload to Google Drive. (We can charge for up to 2TB of Google Drive storage space.)
We provide camera images, Lidar point cloud, RTK-GPS, and Radar tensor (for network input) via Google Drive for flexibility.

<a href="https://drive.google.com/drive/folders/1IfKu-jKB1InBXmfacjMKQ4qTm8jiHrG_?usp=share_link"> Download link </a>

After downloading all the dataset, please arrange the sequence directory with the following structure:
```
├── sequence_number (i.e., 1, 2, ..., 58)
      ├── cam_front
      ├── cam_left
      ├── cam_rear
      ├── cam_right
      ├── description.txt
      ├── info_calib
      ├── info_label
      ├── os1-128
      ├── os2-64
      ├── radar_zyx_cube
```

## Sequence composition

Each of the 58 sequences consists of listed files or folders as follows.
We add the explanation or usage next to each file or folder.
```
Sequence_number.zip (e.g. 1.zip)
├── cam-front: Front camera images
├── cam-left: Left camera images
├── cam-rear: Rear camera images
├── cam-right: Right camera images
├── cell_path: File to generate data from bag or radar ADC files (refer to the matlab file generation code.)
├── description.txt: Required for conditional evaluation (e.g., weather conditions)
├── info_calib: Calibration info btwn 4D radar and Lidar
├── info_label: Label of frames in txt
├── info_label_rev: not used
├── lidar_bev_image: Used for the labeling program
├── lidar_bev_image_origin: not used
├── lidar_point_cloud: Used for the labeling program
├── os1-128: High resolution mid-range Lidar point cloud
├── os2-64: Long-range Lidar point cloud (mainly used)
├── radar_bev_image: Used for the labeling program
├── radar_bev_image_origin: not used
├── radar_tesseract: 4D (DRAE) Radar tensor (250MB per frame) 
├── radar_zyx_cube: 3D (ZYX) Radar tensor (used for Sparse tensor generation)
├── time_info: Used for calibration
```

## Requirements

1. Clone the repository
```
git clone https://github.com/kaist-avelab/K-Radar.git
cd K-Radar
```

2. Create a conda environment
```
conda create -n kradar python=3.8.13 -y
conda activate kradar
```

3. Install PyTorch (We recommend pytorch 1.11.0.)

4. Install the dependencies
```
pip install -r requirements.txt
```

5. Build packages for Rotated IoU
```
cd utils/Rotated_IoU/cuda_op
python setup.py install
```

6. Modify the code in packages
```
Add line 11: 'from .nms import rboxes' for __init__.py of nms module.
Add line 39: 'rrect = tuple(rrect)' and comment line 41: 'print(r)' in nms.py of nms module.
```

7. Build packages for OpenPCDet operations
```
cd ../../../ops
python setup.py develop
```

8. Unzip 'kradar_revised_label_v2_0.zip' in the 'tools/revise_label' directory

We use the operations from <a href="https://github.com/open-mmlab/OpenPCDet">OpenPCDet</a> repository and acknowledge that all code in `ops` directory is sourced from there.
To align with our project requirements, we have made several modifications to the original code and have uploaded the revised versions to our repository.
We extend our gratitude to MMLab for their great work.

## Train & Evaluation
* To train the model, prepare the total dataset and run
```
python main_train_0.py
```

* To evaluate the model, modify the path and run
```
python main_val_0.py (for evaluation)
python main_cond_0.py (for conditional evaluation)
```

* To visualize the inference result, modify the path and run
```
python main_test_0.py (with code)
python main_vis.py (with GUI)
```

## Revising K-Radar Label

There are two primary revisions to our K-Radar label:

1. **Quality Enhancement**: The quality of the K-Radar label has been significantly improved. For instance, the size of the bounding boxes is now better suited to individual objects. Missing labels have been updated using camera images and LiDAR point cloud data. Furthermore, the tracking IDs of the objects have been amended. This revision was made possible by the Information Processing Lab, as acknowledged. We refer to this version of the updated label as v2.0. (The previous one is the label v1.0) You can download it using [the provided link](https://drive.google.com/file/d/1hEN7vdqoLaHZFf7woj5nj4hjpP88f7gU/view?usp=sharing).

2. **Visibility Discrimination**: We now categorize the visibility of each object based on its appearance in LiDAR and 4D Radar.

![image](./docs/imgs/revised_label.png)
An example of a label of an object invisible to Radar but visible to LiDAR illustrated in gray dashed boxes in (a) the front view image, (b) LiDAR point cloud, and (c) 4D Radar heatmap.

Most 4D Radar object detection datasets use LiDAR-detected 3D bounding boxes as reference labels. However, a key difference between LiDAR and 4D Radar sensors is their respective installation locations. In detail, while LiDAR is typically installed on the roof of a vehicle, 4D Radar is placed in front of or directly above the vehicle bumper, as illustrated by the dashed green box in Figure above (a). As a result of this sensor placement disparity, such objects that are detected by LiDAR are not be visible to 4D Radar as shown in Figure above (b,c). This issue leads to the presence of labels that are invisible to Radar in 4D Radar datasets, but visible in LiDAR measurements. Such inconsistencies can negatively impact the performance of 4D Radar object detection neural networks, resulting in a number of false alarms. To address this issue, we distinguish labels that are invisible to Radar and we demonstrate this revision can improve the performance of the 4D Radar object detection network as shown in Model Zoo. We refer to this version of the updated label as v2.1. The label is accessible in the `tools/revise_label` directory.