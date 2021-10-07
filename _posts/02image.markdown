---
layout:     post
title:      "关于PIl和CV2的数据格式"
date:       2021-10-2 21:00:00
author:     "Chai"
header-img: "img/post-bg-2015.jpg"
tags:
    - AI
---

> “你们为什么不能统一下呢”


## Preface
PIL和CV2的数据格式，具体来说是这两个包读取RGB图像后，得到的数据格式。PyTorch在做一般的深度学习图像处理任务时，先使用dataset类和dataloader类读入图片（就我的理解，现在的代码一般也就分成1.dataloader2.model3.train4.test几个重要的部分。）扯远了，在dataset类里，读取图像一般也就是由PIL或CV2完成的，所以如果要自定义数据集，修改dataset类时，就一定会与这两个包接触。本篇blog也就是想要记录我修改dataset时面对这两个包碰到的坑，因为他们读出来的数据格式竟然不一样。

后续可能会补上imageio

参考：https://www.cnblogs.com/ocean1100/p/9494640.html


---

## Part 1
先讲讲PIL，因为我感觉我PIL用得更多一点。

from PIL import Image

import numpy as np

image = Image.open('test.jpg') # 图片是400x300 宽x高 RGB

print (image.size)  # out: (400,300)

image = np.array(image,dtype=np.float32) # image = np.array(image)默认是uint8

print image.shape # out: (400, 400, 3)

可以看到，图片原本是宽X高，Image.open得到的Image.Image.size也是(w,h)，通道数没算进来
而之后np.array得到的output，变成了(w,h,c)，w和h反过来了，并且多了c,像素顺序是RGB。

相对的，对CV2，结果不一样。

import cv2

import numpy as np

image = cv2.imread('test.jpg')

print image.shape # out: (300, 400, 3) (h,w,c) 和skimage类似

cv2.imread得到的结果直接是nparray，而不是PIL那样自己的数据结构，格式为格式为(h,w,c)，像素顺序为BGR。

## Part2
读取图片之后，需要用torchvision的to_tensor()进行之后的训练。
to_tensor()函数接受PIL Image或numpy.ndarray，将其先由HWC转置为CHW格式，再转为float后每个像素除以255（归一化了）。

总结：重点在于匹配图片（读取和to_tensor都会改变顺序）和之后网络的输入的格式（hwc的顺序）。另外如果要对RGB通道单独操作，也需要根据使用的包来调整顺序。
