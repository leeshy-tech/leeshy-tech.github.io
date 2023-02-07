# 目标检测评估指标mAP


## 目标检测任务

目标检测任务是计算机视觉领域的一个分支 ，其关注图片中物体的类别和位置信息，给定一张图片，需要给出图片中感兴趣物体的类别、位置和置信度，位置通过一个矩形的检测框来表示。

![image-20230207225451580](/image/objectionDetection/image-20230207225451580.png)

## 特有名词

ground truth box：人工标注的真实框，相当于标准答案。

BB(bounding box)：预测框

pipeline：流程（流水线）

confidence scores：置信度，用于评估一个预测框中含有某个对象的概率。

## mAP（mean Average Precision）

### IoU 

IoU(intersection over union)直译为交并比（交集除以并集）

![image-20230207215205005](/image/objectionDetection/image-20230207215205005.png)

从图形可以看出，IoU用来衡量两个框（预测框和真实框）之间的相似性。IoU越大，两个框越相似，说明模型预测的越好。

### TP、FN、FP、TN

首先考虑以下的混淆矩阵：

![image-20230207215925441](/image/objectionDetection/image-20230207215925441.png)

与一般的分类问题相比，目标检测任务中True Positives、False Positives、True Negatives和 False Negatives的鉴别会更复杂：

- 对于模型给出的预测框，我们使用置信度阈值来评估一个框是Positive还是Negative。

- 对于Positive预测框，计算与所有ground truth的IoU，取最大值，通过IoU阈值来评估预测为True Positive还是False Positive。
- ground truth的数量减去True Positive即为False Negative的数量。

### Precision、Recall

精度
$$
\text { Precision }=\frac{T P}{T P+F P}
$$
召回率
$$
\text { Recall }=\frac{T P}{T P+F N}
$$
recall和precision是模型性能两个不同维度的度量：

- recall度量的是「查全率」，所有的正样本是不是都被检测出来了。比如在肿瘤预测场景中，要求模型有更高的recall，不能放过每一个肿瘤。

- precision度量的是「查准率」，在所有检测出的正样本中是不是实际都为正样本。比如在垃圾邮件判断等场景中，要求有更高的precision，确保放到回收站的都是垃圾邮件。

这两个参数是矛盾的，二者不可能同时到达最大值，将recall-precision分别做横纵轴绘制图像，图像下面积即为AP（average precision）。

mAP（mean Average Precision）为全类平均正确率，对所有类别的AP取平均值即为mAP。衡量模型在一个数据集上的所有类别的识别性能。

![image-20230207224925877](/image/objectionDetection/image-20230207224925877.png)

