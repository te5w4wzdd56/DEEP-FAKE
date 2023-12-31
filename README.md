![parallel coordinates plot](/images/parcoords.gif)

Read the technical deep dive: https://www.dessa.com/post/deepfake-detection-that-actually-works

# Visual DeepFake Detection

In our recent [article](https://www.dessa.com/post/deepfake-detection-that-actually-works), we make the following contributions:
* We show that the model proposed in current state of the art in video manipulation (FaceForensics++) does not generalize to real-life videos randomly 
collected 
from Youtube.
* We show the need for the detector to be constantly updated with real-world data, and propose an initial solution in hopes of solving deepfake video detection.

Our Pytorch implementation, conducts extensive experiments to demonstrate that the datasets produced by Google and detailed in the FaceForensics++ 
paper are not sufficient for making neural networks generalize to detect real-life face manipulation techniques. It also provides a current solution for such
 behavior which relies on adding more data. 
 
Our Pytorch model is based on a pre-trained ResNet18 on Imagenet, that we finetune to solve the deepfake detection problem.
We also conduct large scale experiments using Dessa's open source scheduler + experiment manger [Atlas](https://github.com/dessa-research/atlas).

## Setup 

## Prerequisities
To run the code, your system should meet the following requirements: RAM >= 32GB , GPUs >=1

## Steps

0. Install [nvidia-docker](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0))
00. Install [ffmpeg](https://www.ffmpeg.org/download.html) or `sudo apt install ffmpeg`
1. Git Clone this repository.
2. If you haven't already, install [Atlas](https://github.com/dessa-research/atlas).
3. Once you've installed Atlas, activate your environment if you haven't already, and navigate to your project folder.

That's it, You're ready to go!

## Datasets
Half of the dataset used in this project is from the [FaceForensics](https://github.com/ondyari/FaceForensics/tree/master/dataset) deepfake detection dataset.
. 

To download this data, please make sure to fill out the [google form](https://github.com/ondyari/FaceForensics/#access) to request access to the data.

For the dataset that we collected from Youtube, it is accessible on [S3](ttps://deepfake-detection.s3.amazonaws.com/augment_deepfake.tar.gz) for download.

To automatically download and restructure both datasets, please execute:

```
bash restructure_data.sh faceforensics_download.py
```

Note: You need to have received the download script from FaceForensics++ people before executing the restructure script.

Note2: We created the `restructure_data.sh` to do a split that replicates our exact experiments avaiable in the UI above, please feel free to change the 
splits as you wish.

## Walkthrough

Before starting to train/evaluate models, we should first create the docker image that we will be running our experiments with. To do so, we already prepared
 a dockerfile to do that inside `custom_docker_image`. To create the docker image, execute the following commands in terminal:
 
 ```
 cd custom_docker_image
 nvidia-docker build . -t atlas_ff
 ```
 
Note: if you change the image name, please make sure you also modify line 16 of `job.config.yaml` to match the docker image name.

Inside `job.config.yaml`, please modify the data path on host from `/media/biggie2/FaceForensics/datasets/` to the absolute path of your `datasets` folder.

The folder containing your datasets should have the following structure:

```
datasets
├── augment_deepfake        (2)
│   ├── fake
│   │   └── frames
│   ├── real
│   │   └── frames
│   └── val
│       ├── fake
│       └── real
├── base_deepfake           (1)
│   ├── fake
│   │   └── frames
│   ├── real
│   │   └── frames
│   └── val
│       ├── fake
│       └── real
├── both_deepfake           (3)
│   ├── fake
│   │   └── frames
│   ├── real
│   │   └── frames
│   └── val
│       ├── fake
│       └── real
├── precomputed             (4)
└── T_deepfake              (0)
    ├── manipulated_sequences
    │   ├── DeepFakeDetection
    │   ├── Deepfakes
    │   ├── Face2Face
    │   ├── FaceSwap
    │   └── NeuralTextures
    └── original_sequences
        ├── actors
        └── youtube
```

Notes:
* (0) is the dataset downloaded using the FaceForensics repo scripts
* (1) is a reshaped version of FaceForensics data to match the expected structure by the codebase. subfolders called `frames` contain frames collected using 
`ffmpeg`
* (2) is the augmented dataset, collected from youtube, available on s3.
* (3) is the combination of both base and augmented datasets.
* (4) precomputed will be automatically created during training. It holds cashed cropped frames.

Then, to run all the experiments we will show in the article to come, you can launch the script `hparams_search.py` using:

```bash
python hparams_search.py
```

## Results

In the following pictures, the title for each subplot is in the form `real_prob, fake_prob | prediction | label`.

#### Model trained on FaceForensics++ dataset

For models trained on the paper dataset alone, we notice that the model only learns to detect the manipulation techniques mentioned in the paper and misses 
all the manipulations in real world data (from data)

![model1](/images/model1.png)
![model11](/images/model11.png)

#### Model trained on Youtube dataset

Models trained on the youtube data alone learn to detect real world deepfakes, but also learn to detect easy deepfakes in the paper dataset as well. These 
models however fail to detect any other type of manipulation (such as NeuralTextures).

![model2](/images/model2.png)
![model22](/images/model22.png)

#### Model trained on Paper + Youtube dataset

Finally, models trained on the combination of both datasets together, learns to detect both real world manipulation techniques as well as the other methods 
mentioned in FaceForensics++ paper. 

![model3](/images/model3.png)
![model33](/images/model33.png)

for a more in depth explanation of these results, please refer to the [article](https://www.dessa.com/post/deepfake-detection-that-actually-works) we published. More results can be seen in the 
[interactive UI](http://deepfake-detection.dessa.com/projects)

## Help improve this technology

Please feel free to fork this work and keep pushing on it.

If you also want to help improving the deepfake detection datasets, please share your real/forged samples at foundations@dessa.com.

## LICENSE
© 2020 Square, Inc. ATLAS, DESSA, the Dessa Logo, and others are trademarks of Square, Inc. All third party names and trademarks are properties of their respective owners and are used for identification purposes only.