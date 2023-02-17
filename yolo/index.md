# YOLO v1深度解读

# You Only Look Once: Unified, Real-Time Object Detection

## 概况

[You Only Look Once: Unified, Real-Time Object Detection](https://www.cv-foundation.org/openaccess/content_cvpr_2016/html/Redmon_You_Only_Look_CVPR_2016_paper.html)

CVPR 2016

## 训练阶段

- 将一幅图像分为7×7（S×S）个网格（grid cell），如果某个物体的中心落在某个网格内，就由该网格预测该物体。
- 每个网格预测 B=2 个bounding box（BB），每个BB回归自身的坐标(x, y, w, h)和置信度confidence共5个值。置信度$ {Pr}(\text { Object }) * \mathrm{IOU}_{\text {pred }}^{\text {truth }}$反映BB含有对象的概率以及预测准确性，为0或IOU。
- 每个网格预测 C 个类别条件概率${Pr}\left(\text { Class }_{i} \mid \text { Object }\right)$。
- 输出维度为S×S×（5*B + C），对于PASCAL VOC数据集，输入448×448×3通道的彩色图像，输入7×7×30的张量。

![image-20230217133437915](/image/objectionDetection/image-20230217133437915.png)

### 网络结构

通过一个深度卷积神经网络抽取图像特征，模型参考GoogleNet

![image-20230217132610274](/image/objectionDetection/image-20230217132610274.png)

输入是448×448×3通道的彩色图像，经过若干卷积层、池化层变成7×7×1024的feature map，经全连接层回归得到7×7×30的张量。

### 损失函数

![image-20230217151158958](/image/objectionDetection/image-20230217151158958.png)

![image-20230217155227050](/image/objectionDetection/image-20230217155227050.png)

## 预测阶段

用模型进行预测，再进行后处理。

![image-20230217140842894](/image/objectionDetection/image-20230217140842894.png)

模型预测出98个BB，以及每个BB的类别全概率（计算得到）：

![image-20230217155242252](/image/objectionDetection/image-20230217155242252.png)

它代表BB含有某个类的概率以及BB与对象的匹配程度。

对于特定类别：

- 将小于设定阈值的概率置0，并将所有BB按概率排序，最后进行NMS（Non-maximum suppression，非极大值抑制）

![image-20230217141017169](/image/objectionDetection/image-20230217141017169.png)

非极大值抑制：

1. 对于概率最大的BB，计算所有BB与之的IOU，去除IOU大于某设定阈值的BB（概率置0）。（IOU大于某一阈值，就认定两个BB是预测了同一个物体，所以这一步的意义是去除重复的预测框）
2. 对余下的概率最大的BB，重复以上过程。

![image-20230217141408828](/image/objectionDetection/image-20230217141408828.png)

## 实验结果

1. 在同准确率上是最快的，在同速度下是最准的，达到了实时性。
2. 定位错误最多，背景错误比R-CNN少很多，这是因为YOLO输入整张图片，能看到更大范围的图像。
3. 对于集成模型，与R-CNN继承能进一步提升准确率，但是速度没有提升。
4. 泛化性能好，在现实数据集训练的结果在艺术作品上也表现的很好，远超其他方法。

## 参考

[【精读AI论文】YOLO V1目标检测，看我就够了](https://www.bilibili.com/video/BV15w411Z7LG?p=4&vd_source=c421bf5ed1b524a57c89542f6c02aceb)


