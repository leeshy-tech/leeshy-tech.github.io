# YOLO v2深度解读


# YOLO9000: Better, Faster, Stronger

## 概况

[YOLO9000: Better, Faster, Stronger](https://openaccess.thecvf.com/content_cvpr_2017/html/Redmon_YOLO9000_Better_Faster_CVPR_2017_paper.html)

CVPR 2017

## Better

### Batch Normalization

![image-20230218140208699](/image/objectionDetection//image-20230218140208699.png)

### High Resloution Classifier

高分辨率分类器

YOLO v1骨干网络是在224×224图像上训练，再运用到448×448图像的检测。YOLO v2使用448×448图像进行训练。

![image-20230218140552450](/image/objectionDetection//image-20230218140552450.png)

![image-20230218140743015](/image/objectionDetection//image-20230218140743015.png)

### Anchor / Dimension Clusters / Direct location prediction

预设一组锚框（anchor boxes），使得不同大小的锚框负责预测不同尺寸的物体。

![image-20230218141156256](/image/objectionDetection//image-20230218141156256.png)

每个grid cell产生5个anchor，类别信息改为由anchor预测，每个anchor带有4个位置参数，一个置信度参数和20个类别参数。

![image-20230218141449494](/image/objectionDetection//image-20230218141449494.png)

anchor的5个尺寸由对COCO和Pascal VOC数据集聚类得到。

![image-20230218141905508](/image/objectionDetection//image-20230218141905508.png)

在anchor坐标回归上，添加sigmoid函数，将BB中心始终约束在grid cell内。

![image-20230218142508059](/image/objectionDetection//image-20230218142508059.png)

### 损失函数

![image-20230218142939440](/image/objectionDetection//image-20230218142939440.png)

### Fine-Grained Features

细粒度特征，把浅层网络的特征与深层网络特征叠加，整合不同尺度的特征。

![image-20230218143650118](/image/objectionDetection//image-20230218143650118.png)

![image-20230218144034581](/image/objectionDetection//image-20230218144034581.png)

### Multi-Scale Training

输入不同尺度的图像，迫使模型学习到不同尺度的特征。输入大尺度图像，模型速度变慢，精度提高。对于小尺度图片，速度变快，精度降低。

![image-20230218144245549](/image/objectionDetection//image-20230218144245549.png)

![image-20230218144430520](/image/objectionDetection//image-20230218144430520.png)

### 所有trick的提升

加入anchor虽然减小了mAP，但是recall大大提升。

![image-20230218144515821](/image/objectionDetection//image-20230218144515821.png)

## Faster

采用Darknet19作为骨干网络

![image-20230218144713478](/image/objectionDetection//image-20230218144713478.png)

整体结构：

![image-20230218145006603](/image/objectionDetection//image-20230218145006603.png)

### Stronger

检测类别变得更多：将COCO目标检测数据集和Imagenet分类数据集联合训练。

![image-20230218145154801](/image/objectionDetection//image-20230218145154801.png)

## 参考

[【精读AI论文】YOLO V2目标检测算法](https://www.bilibili.com/video/BV1Q64y1s74K/?spm_id_from=333.999.0.0&vd_source=c421bf5ed1b524a57c89542f6c02aceb)

