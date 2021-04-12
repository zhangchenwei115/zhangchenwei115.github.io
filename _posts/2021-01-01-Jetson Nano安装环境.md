---
layout:     post
title:      Jetson Nano安装环境 
subtitle:   如何在Jetson Nano上运行3d姿态检测模型
date:       2021-01-01 
author:     Chenwei
header-img: img/tensorflow.jpg
catalog: true
tags:
    - 机器学习
    - Jetdon Nano
---


# 组装及烧录系统
上手第一步需要烧录系统，在官网上有详细的教程，Nano的系统里自带了cuda，opencv，以及python2和3。**注意**，只写python则代表调用的是python2，需要用python3时请记得加上3。用的usb 5v2A电源供电，但经常提示电压不足，后续可能会考虑换上DC电源。
## 所用摄像头
注意此处为重点，一般情况下，在pc上使用的都是电脑自带摄像头或者usb摄像头，此摄像头用opencv可直接调取，cv2.Capture（0）。但是我们在jetson nano上所使用的摄像头为CSI摄像头。两者对比如下：  
**USB摄像头：**  
· 优：很容易整合。   
· 优：可以做很多的离线的图像工作（曝光控制，帧率等）。   
· 优：提供输入/中断功能，可为您节省计算应用程序时间（例如，在新帧上中断）。   
· 缺：由于USB总线使用CPU时间，如果使用高的CPU占用，这会影响您的应用程序。   
· 缺：对于使用硬件视觉管线（硬件编码器等）不是最佳的。   
· 优：可以长距离工作（最高可达USB标准）  。 
· 优：可以支持更大的图像传感器（1英寸或更高，以获得更好的图像质量和更少的噪音）  

**CSI摄像头：**  
· 优：根据CPU和内存使用情况进行优化，以便将图像处理并存入内存。   
· 优：可以充分利用硬件的视觉管线。   
· 缺：支持较短距离（通常不超过10cm）。除非您使用序列化系统（GMSL，FPD Link，COAXPress，Ambarella） ，但这些系统目前尚不成熟并且需要定制。   
· 缺：通常与手机相机模块的小型传感器一样，要不就得多花点钱去定制。通过TX1/2中的硬件去噪降低小传感器的额外噪音。   
· 优：可执行底层访问与控制传感器/摄像头。
  
    
## 环境设置  
第一步需要设置好环境，比较繁琐，所以特此记录一下：  
1. 虽然系统自带cuda cudnn，但是并没有加入环境变量，需要手动添加  
```bash
sudo gedit  ~/.bashrc   #打开bash文件
在最后添加
export CUBA_HOME=/usr/local/cuda-10.0
export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64:$LD_LIBRARY_PATH
export PATH=/usr/local/cuda-10.0/bin:$PATH
！！！重点，我的镜像版本可能比较新，所以我的cuda是10.2，股上面要改成10，2
然后
source ~/.bashrc #运行一下
and then we can see cuda installed
nvcc -V
```
2. 安装pip3
```bash
sudo apt-get install python3-pip python3-dev
python3 -m pip install --upgrade pip  #升级pip
sudo gedit /usr/bin/pip3   #打开pip3文件
```
将原来的
```bash
from pip import main
if __name__ == '__main__':
    sys.exit(main())
```
改为
```bash
from pip import __main__
if __name__ == '__main__':
    sys.exit(__main__._main())
```
安装成功，测试 pip3 -V  

3. 安装 pytorch
按照官方提示，安装最新版本pytorch，以及torchvision。[网址](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-7-0-now-available/72048)

4. 安装torch2trt
[网址]( https://gilberttanner.com/blog/run-pytorch-models-on-the-jetson-nano-with-tensorrt)
```bash
git clone https://github.com/NVIDIA-AI-IOT/torch2trt
cd torch2trt
sudo python3 setup.py install
```
# 运行3D人体姿态检测模型
我们所使用的程序是来自[lightweight openpose](https://github.com/Daniil-Osokin/lightweight-human-pose-estimation.pytorch)，此代码把openpose的网络结构精简并且换位Mobilnet，使得速度加快并且精度仅仅损失一点点（有待后续测试）  
下面尝试运行  
```bash
https://github.com/Daniil-Osokin/lightweight-human-pose-estimation-3d-demo.pytorch.git #克隆到本地
```
注意因为原作者使用的是usb摄像头，所以--video 0，即可直接调取摄像头获取结果，但是我们使用的是CSI摄像头，必须通过解码什么社么（具体原理不太懂）才可以运行，所以需要把demo.py和moduls里的代码改动，具体原理为原代码为
```bash
cv2.VideoCapture(0)
需要把0改为：
nvcamerasrc ! video/x-raw(memory:NVMM), width=1920, height=(int)1080,\
 format=(string)I420, framerate=(fraction)30/1 ! \
 nvvidconv flip-method=2 ! video/x-raw, format=(string)BGRx ! \
 videoconvert ! video/x-raw, format=(string)BGR ! appsink
 ```
 稍改原代码结构，即可实现运行。
 