---
layout:     post
title:      Tensorflow基本操作学习
subtitle:   Tensorflow的基本操作和环境配置
date:       2020-11-29
author:     Chenwei
header-img: img/tensorflow.jpg
catalog: true
tags:
    Tensorflow
---


# Tensorflow
## Tensorflow的第一次配置
因为我的电脑Thinkpad X1 2017内存太小而且我主要用的是winddows系统，所以我拿出我封尘五六年的老电脑（acer，显卡 gtx 640m，1Tb机械+128固态），于是我给这台老电脑装了linux，然后用ssh，在我的ThinkPad上面操作，具体图片如下  
![picture1](/img/IMG_6386.jpg)  

首先用putt 192.168.2.159 ；连上linux， ubuntu 20.02。然后开始[配置环境](https://www.tensorflow.org/install/pip?hl=zh-cn)  
首先创建虚拟环境，意义在于和系统环境隔离开，每个项目可以有自己的环境，安装各种东西互不影响  

**创建激活虚拟环境**
```bash
python3 -m venv --system-site-packages ./venv
source ./venv/bin/activate  # 激活
deactivate #退出
```
