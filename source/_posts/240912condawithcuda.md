---
title: 使用conda安装cuda
tags:
  - conda
  - cuda
categories:
  - - 教程
excerpt: 使用conda安装cuda，避免cuda版本不匹配的问题
date: 2024-09-12 10:02:27
---

# 使用conda安装cuda
因为需要在本地跑很多项目，有些项目在安装的时候会出现很多猝不及防的问题

最大的问题就是在正常创建conda环境，正常安装torch之后，一些依赖在安装的时候需要安装cuda,如nvcc，这时如何安装匹配的cuda版本就是一个问题

为了避免安装某个版本的cuda后，在跑其他项目的时候出现cuda版本不匹配的问题，我们可以使用conda安装cuda

## 安装cuda
> 注意，网上有教程说当需要使用conda安装包时，先将所有需要conda安装的包都安装好，再安装pip安装的包（虽然我没遇到过类似情况）

参考[官方文档](https://docs.nvidia.com/cuda/archive/12.1.0/cuda-installation-guide-linux/index.html#conda-installation)

```bash
conda install cuda -c nvidia/label/cuda-12.1.0
```

避坑，使用pip安装cuda识别不到nvcc，很怪。同时conda安装11.8时无法下载，再测试一下情况。

>其实能装cuda-11.8.0，只不过一开始不会显示进度，等一会就好了，可以用nload看一下下载速度，就不会焦虑了

>相似的问题，使用pip安装需要编译的时候，有时候也很慢，为了避免焦虑可以使用-v参数，显示详细信息

同时
>避坑！！
    
```bash
pip install nvidia-pyindex #不要复制运行
```
会直接覆盖掉你的pip.conf文件，更改pip默认配置，不建议下载

[相关问题讨论](https://github.com/NVIDIA/tensorflow/issues/98)

如果已经不小心覆盖率，解决办法：

```bash
pip config debug
```
找到所有的配置文件，删除掉就行了，或者将对应配置使用unset参数

## 其他问题

在编译mmcv的时候，会出现找不到c++的问题，但实际上已经安装了g++
```bash
which g++ # /usr/bin/g++
which c++ #c++ not found
```
解决办法：
```bash
sudo ln -s /usr/bin/g++ /usr/bin/c++
```
这样就可以解决问题了

由此还想到一个问题，就是g++/gcc的版本控制，因为发行版会默认对应一个版本，但是有时候我们需要使用其他版本，这时候就需要更改默认版本

```bash
#以下两行仅供示意，未经过验证
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 100
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 100
```
反正就是使用update-alternatives来更改默认版本，这样就可以解决问题了

[参考链接](https://askubuntu.com/questions/26498/how-to-choose-the-default-gcc-and-g-version)


