# Radar-Image Association Network for 3D Object Detection (RADIANT)


## Setting up docker
Note: Using docker is an option if your host machine has ubuntu 20.04 OS. If you have OS other than Ubuntu, it is recommended to use docker. If you have Ubuntu in your machine and like to use that instead of docker, proceed [here](#environment-setup)

1. Go through the [Documentation](https://docs.docker.com/desktop/) to install and start the docker engine.

2. Once, docker engine is installed ans started, use the following command to get ubuntu docker image:

```
docker pull ubuntu:<version-number>
```
3. After pulling the ubuntu docker image, make sure the image or container is running. 

4. If the docker container is not running, then use the follwing command to run the container and use it:
```
docker run --gpus all -it --shm-size=16g ubuntu:<tag>
```
5. initially, at first run of docker image, the tag will be the version number used for pulling ubuntu container/image.

6. After running the container, we will notice that the user of the image is named root in the format follows:
```
root@<container-id>:/#
```
7. Here, note the container-id to save all the progress later on.

8. As the container is running and ready to use now, proceed [Here](#environment-setup) to work on.
9. After working, to avoid loss of the progress and save the content worked on, exit the docker using:
```
exit
```
10. Now, start the docker container in which the work was done:
```
docker start <container-id>
```
11. Look of the name of the container by identifying it using the container-id in list of containers. To get the list use command:
```
docker ps
```
12. After noting down the name of container worked on, commit the changes or content in the container using:
```
docker commit <image-name> ubuntu:<tag>
```
13. Here, the tage can be anything of user's choice. But remember the tage to use the container with saved progress in future.

14. After commiting changes, stop the docker using:
```
docker stop <image-name>
```
15. If you wish to have the container to saved to the host machine, use the following command:
```
docker save ubuntu:<tag> > <name>.tar
```
16. To load this file(container), use the follwing command:
```
docker load < <name>.tar
```

Note: The container-id, name of container always change each time we run the container. Only the tag won't change and using different tag name while commiting changes actually create another custom container leaving the one you use aside without deleting.



## Initial setup

1. Run the following commands to get all required tools
```
apt-get update
apt-get install wget
apt-get install git
apt-get install vim
```

2. Run all the following commands for CUDA Setup
```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.1.1/local_installers/cuda-repo-ubuntu2004-11-1-local_11.1.1-455.32.00-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2004-11-1-local_11.1.1-455.32.00-1_amd64.deb
sudo apt-key add /var/cuda-repo-ubuntu2004-11-1-local/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
```

3. Setup torch based on the CUDA version
```
conda install pytorch==1.9.0 torchvision==0.10.0 cudatoolkit=11.1 -c pytorch -c conda-forge
```

4. Create a conda environment and activate it
```bash
conda create -n 3dmot python=3.6
conda activate 3dmot
```

5. Clone the repository
```bash
git clone https://github.com/airou-lab/3dMOT.git
```

6. Install requirements
```bash
cd 3dMOT
pip install -r requirements.txt
```

7. Clone corresponding tools repo
```bash
git clone https://github.com/xinshuoweng/Xinshuo_PyToolbox
```

8. Install requirements of this tools repo
```bash
cd Xinshuo_PyToolbox
pip install -r requirements.txt
cd ..
```

9. Add the tools path to environment variable to use the tools
```bash
export PYTHONPATH=${PYTHONPATH}:/path/to/3DMOT
export PYTHONPATH=${PYTHONPATH}:/path/to/3DMOT/Xinshuo_PyToolbox
```

10. Install requiremnets necessary for nuScenes data operations
```bash
cd scripts/nuScenes
pip3 install -r requirements.txt
cd ../../
```

## Data Preparation
* Please download the official [nuScenes full dataset (v1.0)](https://www.nuscenes.org/download), then uncompress, and put the data under "./data/nuScenes/data" folder in the following structure:

```
3DMOT
├── data
│   ├── nuScenes
│   │   │── data
│   │   |   │── samples
│   │   |   │── sweeps
│   │   |   │── v1.0-mini
│   │   |   │── v1.0-test
│   │   |   │── v1.0-trainval
├── AB3DMOT_libs
├── configs
```

## Implementation

1. Convert the nuScenes data into KITTI format as the code processes data only in KITTI format:

```bash
python scripts/nuScenes/export_kitti.py nuscenes_gt2kitti_trk --split val
python scripts/nuScenes/export_kitti.py nuscenes_gt2kitti_trk --split test
```

The above code will generate nuScenes GT data at "./data/nuScenes/nuKITTI/tracking" following the KITTI format. Please check if the data has the following structure:

```
3DMOT/data
├── nuScenes
│   ├── nuKITTI
│   │   │── tracking
│   │   │   │── produced
│   │   │   │   ├──correspondence & split
│   │   │   │── test
│   │   │   │   ├──calib & image_02 & oxts & velodyne 
│   │   |   │── val
│   │   │   │   ├──calib & image_02 & label_02 & oxts & velodyne 
├── AB3DMOT_libs
├── configs
```

2. Run the following code for 3D Multi-Object Tracking
```bash
python main.py --dataset nuScenes --det_name megvii --split val
python main.py --dataset nuScenes --det_name centerpoint --split val
```

3. Convert the generated results from KITTI to nuScenes to evaluate
```bash
python scripts/nuScenes/export_kitti.py kitti_trk_result2nuscenes --result_name megvii_val_H1 --split val
python scripts/nuScenes/evaluate.py --result_path ./results/nuScenes/megvii_val_H1/results_val.json

```

4. Visualize the generated results 
```bash
python scripts/post_processing/trk_conf_threshold.py --dataset nuScenes-- result_sha megvii_val_H1
python scripts/post_processing/visualization.py --dataset nuScenes --result_sha megvii_val_H1_thres --split val
```
