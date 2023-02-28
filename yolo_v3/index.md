# YOLO v3深度解读


# YOLOv3: An Incremental Improvement

## 概况

[YOLOv3: An Incremental Improvement](https://arxiv.org/abs/1804.02767)

arxiv 2018

## 算法

### 骨干网络

模型层数变深，并加入了残差连接。

![image-20230218152805888](/image/objectionDetection/image-20230218152805888.png)

Darknet-53实现了精度和速度的折中。

![image-20230218153115427](/image/objectionDetection/image-20230218153115427.png)

输入416*416的图像，骨干网络是一个全卷积网络，去除了全连接层，输出3个尺度的特征图，对应不同数量的grid cell（13 26 52），不同感受野负责不同尺度物体的预测。特征金字塔（Feature Pyramid Networks）思想。

![image-20230218154318822](/image/objectionDetection/image-20230218154318822.png)

![image-20230218155044103](/image/objectionDetection/image-20230218155044103.png)

### 损失函数

![image-20230218155618227](/image/objectionDetection/image-20230218155618227.png)


