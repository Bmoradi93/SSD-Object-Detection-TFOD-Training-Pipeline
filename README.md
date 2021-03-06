# SSD-Object-Detection-TFOD-Pipeline
This repository uses Tensorflow Object Detection(TFOD) to implement a training pipeline for multiple object detection applications based on Single Shot Multibox Detector(SSD). The pipeline is trained and tested on a regular vehicle dataset. 

![Selection_104](https://user-images.githubusercontent.com/47978272/143666316-8d11f028-fbb6-446f-9668-32f39d5b8acb.png)

![Selection_107](https://user-images.githubusercontent.com/47978272/143666457-299d1921-3508-411a-9d69-fbe1c1b6c6d0.png)


The entire training and inference process takes place in a docker container and the Dockerfile is provided to pull and build. The Dockerfile is based on TFOD docker image. The main branch of this repository uses Tensorflow 1.x. It is recommended to use a GPU-Powered machine for training process. 
An SSD model is able to operate in real-time(up to 59 fps on 300x300pxl input images).  

## Setup
### Docker Engine Installation
Please following this [instructions](https://docs.docker.com/engine/install/ubuntu/) to install docker engine on your ubuntu 18/20.04 machine.
It is highly recommended to use an NVIDIA-GPU on your machine. In order to use your GPU in your docker image, you must install [nvidia-docker](https://cnvrg.io/how-to-setup-docker-and-nvidia-docker-2-0-on-ubuntu-18-04/). You only need to do Step #2. 

### Pull the docker image
A basic docker container is built and pushed into docker hub. Once you have both docker engine and nvidia-docker installed on your ubuntu machine, open a terminal and pull the docker image.
```
docker pull behnammoradi026/deep_learning:tfod
```
Note: CUDA is already installed on this docker image.


### RUN the docker container
Once the docker image is pulled, run the following script to run and enter the container:

```
# Navigate to the repository's main directory
cd SSD-Object-Detection-TFOD-Training-Pipeline

# Run and enter the docker container
source scripts/run_docker.sh

```
### Install TFOF dependencies and annotation tools
In order to install essential dependencies for annotation and testing tensorflow object detection (TFOD) API, please run the following script. 
Note: Make sure your terminal is inside docker container!

```
# Navigate to the repository's main directory
cd SSD-Object-Detection-TFOD-Training-Pipeline

# Install dpendencies.
source scripts/install_tfod.sh

```
If your TFOD is properly installed, then you should see the following results on your terminal:

```
[       OK ] ModelBuilderTF1Test.test_invalid_model_config_proto
[ RUN      ] ModelBuilderTF1Test.test_invalid_second_stage_batch_size
[       OK ] ModelBuilderTF1Test.test_invalid_second_stage_batch_size
[ RUN      ] ModelBuilderTF1Test.test_session
[  SKIPPED ] ModelBuilderTF1Test.test_session
[ RUN      ] ModelBuilderTF1Test.test_unknown_faster_rcnn_feature_extractor
[       OK ] ModelBuilderTF1Test.test_unknown_faster_rcnn_feature_extractor
[ RUN      ] ModelBuilderTF1Test.test_unknown_meta_architecture
[       OK ] ModelBuilderTF1Test.test_unknown_meta_architecture
[ RUN      ] ModelBuilderTF1Test.test_unknown_ssd_feature_extractor
[       OK ] ModelBuilderTF1Test.test_unknown_ssd_feature_extractor
----------------------------------------------------------------------
Ran 21 tests in 0.135s

OK (skipped=1)
```


## Preparing the dataset(records and classes)
### Annotation
There are several tools for image annotation. The best one is labelImg which is installed on the docker image.
Note: SSD-Object-Detection-TFOD-Training-Pipeline is shared between your host and the docker image. Whatever you paste or generate inside that folder will remain after the docker image is down.
To start annotating data:
1- Split your dataset into training data (90%) and testing data (10%).
2- Create your project folder in dataset dirrectory:
```
cd SSD-Object-Detection-TFOD-Training-Pipeline/dataset

mkdir <your project name>

cd <your project name>

mkdir train test
```
3- Paste your training and testing images in their related directories.
4- In your docker terminal, open labelImg and start annotating your training and testing data.
```
cd SSD-Object-Detection-TFOD-Training-Pipeline# cd dataset/<your project name>/annotations/train

# Run the annotation tool
labelImg
```
5- start annotating your dataset:

![Selection_113](https://user-images.githubusercontent.com/47978272/143763292-70640868-6c4d-4a1a-9b20-64dcfc914158.png)

Note: Working with labelImg is simple and straightforward. This [tutorial](https://www.youtube.com/watch?v=Tlvy-eM8YO4) explains the process.
Note: Make sure you run labelImg inside the docker container.

### Generating records and label map
TFOD uses the record files to read training and testing images and annotations.
Once your annotation process (for both training and testing images) is done, then generate the record files:

```
# Navigate to the repository's main directory
cd SSD-Object-Detection-TFOD-Training-Pipeline

export PROJECT_NAME=<your project name>

# Generate your record files
cd SSD-Object-Detection-TFOD-Training-Pipeline/scripts

source run_generate_tfrecords.sh
```

The label map file is simple to generate.

```
cd SSD-Object-Detection-TFOD-Training-Pipeline/dataset/<your project name>/records

touch classes.pbtxt
```

I assume you have one label for your dataset. Open classes.pbtxt and paste the followings:

```
item {
    id: 1
    name: '<your label>'
}

```
Note: replace "<your label>" with your label name.
Note: If you have more than one label (for example two labels), then you must add another item i your classes.pbtxt file:
    
```
item {
    id: 1
    name: '<your label #1>'
}

item {
    id: 2
    name: '<your label #2>'
}
```

## Pre-trained Model and Network Configuration
### Pre-trained model
In order to start training process, you must download [ssd_inception_v2](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf1_detection_zoo.md):

```
cd SSD-Object-Detection-TFOD-Training-Pipeline/workspace/<your project name>/training

mkdir exported_model training

cd training

wget http://download.tensorflow.org/models/object_detection/ssd_inception_v2_coco_2018_01_28.tar.gz

tar -zxvf ssd_inception_v2_coco_2018_01_28.tar.gz
```


### Network Configuration

```
cd SSD-Object-Detection-TFOD-Training-Pipeline/network_configuration/

mkdir <your project name>

cd <your project name>

# You can use the template inside vehicle_detection project.
cp ../vehicle_detection/ssd_hypes.config ./

# Open the file with your text editor and replace the <YOUR PROJECT TITLE> with your own project name in the entire file.
```

## Training
Once your configuration file is ready, you can start the training process:

```
# Navigate to the repository's main directory
cd SSD-Object-Detection-TFOD-Training-Pipeline

export PROJECT_NAME=<your project name>

source scripts/train.sh
```

Once the training process is done, you must generate your model from the generated checkpoints:

```
# Navigate to the repository's main directory
cd SSD-Object-Detection-TFOD-Training-Pipeline

export PROJECT_NAME=<your project name>

# Open the scripts/export_model.sh and meke sure your pointing to the right TRAINED_CHECKPOINT_PREFIX

source scripts/export_model.sh

```
Your frozen_inference_graph.pb is saves inside SSD-Object-Detection-TFOD-Training-Pipeline/workspace/<your project name>/exported_model

## Inference
```
# Navigate to the repository's main directory
cd SSD-Object-Detection-TFOD-Training-Pipeline

# Open scripts/inference.sh and edit the RAW_IMAGE(Enter your own name at the end of the line)
# Save the file

export PROJECT_NAME=<your project name>

source scripts/inference.sh

# The output is saved in this directory! 

```

## Acknowledgment
[Tensorflow Object detection repository](https://github.com/tensorflow/models)
    
[Single Shot Multibox Detecor Paper](https://arxiv.org/abs/1512.02325)
    
[TFOD documentation](https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/training.html)
