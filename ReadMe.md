# HSR workspace

Author: [Tobit Flatscher](https://github.com/2b-t) (January 2023), Edited by [Samuel Sze] (April 2023) for use in team ORIon

[![Tests](https://github.com/ori-programme-grant/hsr_ws/actions/workflows/build.yml/badge.svg)](https://github.com/ori-programme-grant/hsr_ws/actions/workflows/build.yml)


## 0. Overview

This repository contains a Docker workspace for working with the [Toyota HSR](https://www.toyota-europe.com/innovation/mobility-solutions/robotics) robot in combination with the ORI toolset.



## 1. Set-up

This workspace is build around a Docker container but feel free to use it without it. For latter purpose please refer to the [`docker/Dockerfile`](./docker/Dockerfile) for the required dependencies. The basic set-up and configuration with Visual Studio code is discussed in the documentation of [this repository](https://github.com/2b-t/docker-realtime/tree/main/doc/docker_basics) in more detail.

Currently some of the files use [`git-lfs`](https://github.com/git-lfs/git-lfs). Before proceeding with the next step make sure that you have `git-lfs` installed.

```bash
$ sudo apt-get install git-lfs
```

Else when launching the stack this might result in an error `"Unsupported image format"` when starting the navigation stack. In this case it tries to access the `pgm` file which is set to be stored compressed.

After cloning this repository you will have to update its dependencies. For version control we will use [VCS tool](http://wiki.ros.org/vcstool) instead of Git submodules. In order to **pull the latest repositories** please import them with the following command (either inside the Docker or on the host):

```bash
$ cd src
$ vcs import < .repos --recursive
```

This should clone the desired sub-repositories into your workspace using your SSH credentials. They are excluded in the [`.gitignore`](./.gitignore) file so that they will not be part of your commits.

```

The **network configuration** is done over the [`docker/.env`](./docker/.env) file:

```bash
HSR_IP="192.168.94.121"
HSR_HOSTNAME="hsr"
YOUR_IP="192.168.94.20"
```

There as a first step you will have to **set the IP of the HSR as well as your own IP** (see  `$ ifconfig`).

If you want to run it in simulation set it to

```bash
HSR_IP="127.0.0.1"
HSR_HOSTNAME="hsrb"
YOUR_IP="127.0.0.1"
```

Then either **run the Docker** manually with

```bash
$ docker compose -f docker/docker-compose.yml build 
```

(or its corresponding alternative for Nvidia graphics cards [running the Nvidia driver](https://nvidia.github.io/nvidia-container-runtime/) `docker/docker-compose-gui-nvidia.yml`). 

Or **run the Docker** in vscode code (recommended) with: 

Alternatively use the corresponding [Visual Studio Code Dev Containers integration](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) as described [here](https://github.com/2b-t/docker-realtime/blob/main/doc/docker_basics/VisualStudioCodeSetup.md). In latter case which of the two said Docker-Compose files is used can be adjusted in [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json).

```bash
$ docker compose
```
1. With dev container installed, press the green button in the lower left corner of vscode
2. Select reopen in container
3. ctrl-shift-p and type rebuild - rebuild without cache will rebuild the entire docker container, whereas rebuild cache will take in previously built code in Dockerfile. 

## 2. Running
After the docker image is built, go into the container and complete these steps:

Build caffe
```bash
$ cd caffe
$ mkdir build && cd build
$ cmake -DCPU_ONLY=ON -DCMAKE_INSTALL_PREFIX=/usr ..
$ make 
$ sudo make install
```

Build gpg
```bash
$ cd gpg
$ mkdir build && cd build
$ cmake ..
$ make
$ sudo make install
```
Fixing gpd build error by adding hints to findcaffe.cmake

Go to **/orion_ws/src/gpd/FindCaffe.cmake**, replace 


    find_path(Caffe_INCLUDE_DIR NAMES caffe/caffe.hpp caffe/common.hpp caffe/net.hpp caffe/proto/caffe.pb.h caffe/util/io.hpp caffe/util/db.hpp)

with

    find_path(Caffe_INCLUDE_DIR NAMES caffe/caffe.hpp caffe/common.hpp caffe/net.hpp caffe/proto/caffe.pb.h caffe/util/io.hpp caffe/util/db.hpp
    HINTS
    /usr/include/)

Replace

    find_library(Caffe_LIBS NAMES caffe)

with

    find_library(Caffe_LIBS NAMES caffe
    HINTS
    /usr/include/)

Install software dependencies of the source packages by using rosdep from the orion_ws base directory:
    rosdep install --from-paths src --ignore-src -r -y

Check your outstanding dependencies:
    rosdep check --from-paths src --ignore-src -r -y

Install nltk resources

    python3 src/orion_speech_recognition/orion_asr/scripts/download_nltk_resources.py

Setup your environment

    # do this once, when you first set up your environment
    cd ~/orion_ws
    mkdir db

Proceed to **build the workspace** and source it

```bash
$ catkin build
```

In order to be able to run **graphical user interfaces** from inside the Docker you will have to type

```bash
$ xhost +local:root
```

on the **host system**.

## 3. Known issues

Error: point_cloud_filtering/DetectSurface.h: No such file or directory

Go to ~/orion_ws/src/orion_manipulation/point_cloud_filtering/include/point_cloud_filtering/surface_segmenter.h
Comment out Line 29 #include "point_cloud_filtering/DetectSurface.h"
Run catkin make again
You should now encounter error that say DetectSurface is missing.
Uncomment Line 29 #include "point_cloud_filtering/DetectSurface.h"
Run catkin make again

It is most likely that this error has something to do with the order that the packages are initialised in catkin. More testing is needed.

If everything worked out correctly you should be able to see both deputies, your PC (`localhost`) as well as the HSR online and updating regularly, like shown in the top right of the screenshot below.

