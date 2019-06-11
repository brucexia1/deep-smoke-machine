# deep-smoke-machine
Deep learning for smoke detection. The videos are from the [smoke labeling tool](https://github.com/CMU-CREATE-Lab/video-labeling-tool). The code in this repository assumes that Ubuntu 18.04, Nvidia drivers, cuda, and cuDNN are installed.

Clone this repository and set the permission.
```sh
git clone --recursive https://github.com/CMU-CREATE-Lab/deep-smoke-machine.git
sudo chown -R $USER deep-smoke-machine/
sudo addgroup [group_name]
sudo usermod -a -G [group_name] [user_name]
groups [user_name]
sudo chmod -R 775 deep-smoke-machine/
sudo chgrp [group_name] deep-smoke-machine/
```
Create conda environment and install packages. It is important to install pip first inside the newly created conda environment.
```sh
conda create -n deep-smoke-machine
conda activate deep-smoke-machine
conda install pip
which pip # make sure this is the pip inside the deep-smoke-machine environment
sh deep-smoke-machine/back-end/install_packages.sh
```
Update the optical_flow submodule.
```sh
cd deep-smoke-machine/back-end/www/optical_flow/
git submodule update --init --recursive
git checkout master
```
Install PyTorch.
```sh
# For cuda 9.0
conda install pytorch torchvision cudatoolkit=9.0 -c pytorch

# For cuda 10.0
conda install pytorch torchvision cudatoolkit=10.0 -c pytorch
```
Obtain user token from the [smoke labeling tool](https://smoke.createlab.org/gallery.html) and put the user_token.js file in the deep-smoke-machine/back-end/data directory. You need permissions from the system administrator to download the user token. After getting the token, get the video metadata.
```sh
python get_metadata.py confirm
```
Split the metadata into three sets: train, validation, and test.
```sh
python split_metadata.py confirm
```
Download all videos in the metadata file.
```sh
python download_videos.py
```
Process all videos into rgb frames and optical flows. Save all data to the disk.
```sh
python process_videos.py
```
Extract [I3D features](https://github.com/piergiaj/pytorch-i3d).
```sh
python extract_features.py
```
Train the model with the training and validation sets. Pretrained weights are obtained from the [pytorch-i3d repository](https://github.com/piergiaj/pytorch-i3d).
- [Two-Stream Inflated 3D ConvNet](https://arxiv.org/abs/1705.07750)
- [Two-Stream ConvNet](http://papers.nips.cc/paper/5353-two-stream-convolutional)
```sh
python train.py [method]

# Use I3D features + SVM
python train.py svm

# Use Two-Stream Inflated 3D ConvNet
python train.py i3d

# Use Two-Stream ConvNet
python train.py ts
```
Test the performance of a model on the test set.
```sh
python test.py [method] [model_path]

# Use I3D features + SVM
python test.py svm saved_svm/e5ca667-svm.pkl

# Use Two-Stream Inflated 3D ConvNet
python test.py i3d saved_i3d/99ca217/99ca217-i3d-64022.pt
```
