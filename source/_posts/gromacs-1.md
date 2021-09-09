---
title: Gromacs-1 安装
date: 2019-12-11 21:04:19
tags: 新知
categories: Gromacs
---

## 首先确保安装了最新版的 cmake 以及 gcc 等环境
以ubuntu系统为例：
```
sudo apt install gcc
sudo apt install cmake
```

## 从[官网](http://manual.gromacs.org/documentation/)获得目标版本的源码包并解压(这里以2019.4为例)
```
wget http://ftp.gromacs.org/pub/gromacs/gromacs-2019.4.tar.gz
tar xf gromacs-2019.4.tar.gz
```

## 进入解压后的目录，并创建 build 文件夹并进行 cmake (需联网)
```
cd gromacs-2019.4
mkdir build
cd build
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON
```
这里的 cmake 后面的参数是官方文档的默认参数，另有如下常用参数可选(非全部):
* 添加对 MPI 的支持:
`-DGMX_MPI=on`

* 添加对 NVIDIA CUDA GPU 支持:
`-DGMX_GPU=on`

* 添加对OPENCL的支持(GMX_GPU必须开启):
`-DGMX_USE_OPENCL=on`

* 指定SIMD:
`-DGMX_SIMD=xxx`

## 编译与安装
```
make -j 4
sudo make install
source /usr/local/gromacs/bin/GMXRC
```