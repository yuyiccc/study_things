## [S3FD: Single Shot Scale-invariant Face Detector](https://github.com/yujack333/study_things/blob/master/face-related%20paper/1708.SFD.pdf)
## 此文章为人脸检测算法文章：
### 网络结构：

  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD.png)
  
### 问题：(文章提出了4个问题)
  - 1. 小人脸需要更多的特征来检测，最低层的stride还是太大（8 or 16），文章用的最底层的feature map的stride=4。
  - 2. anchor size应该根据有效感受野来确定而不是理论感受野。因为有效感受野服从正态分布，且只是理论感受野的一部分。
  - 3. anchor size是离散的几个值，而真实的face size是连续的值。之前的匹配方式会导致小人脸和不在anchor size附近的人脸不能被anchor很好的匹配。
  - 4. 底层的feature map由于anchor太多导致大部分都是negative的。这样会导致有很多的假阳性（false positive）。
  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD1.png)

### 方法：
  - 1.文章的backbone用的是VGG，在VGG的基础上再加了额外的几层，具体可以看本文图一。
  
    解决第一个问题的方法就是用更低层的特征图，这样具有更高的分辨率来检测小人脸。
    
  - 2. 解决第二个问题的方法为根据感受野来设计anchor size。
  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD2.png)

  - 3. 第三个问题的解决方法是优化之前的匹配方法。
  
    - 之前的方法：1.每个人脸与之最大iou的anchor匹配。2.计算每个anchor与所有人脸的iou，最大的那个iou如果大于阈值（0.5）则匹配。
    - 改进的方法：1.基于之前的方法，调低阈值（0.35）。2.对于某个face，选取所有与此face大于0.1的anchor，对这些anchor排序，选取前N个anchor与之匹配。
    - 效果：可以大大的提高outer face和tiny face的平均匹配anchor数
      ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD3.png)
      
  - 4. 第四个问题的解决方法：在最低的feature map层的分类预测中background用max-out激活函数。
  
    - ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD4.png)
  
  - 5. 在训练时，采用了hard negtive mining来加速和稳定训练。根据loss来选取negative example，选取loss最大的一些negative来使得正负样本比例为1:3.##
### 结果：
  - ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD5.png)
  - F: 网络结构和anchor大小    S： anchor匹配策略  M：max-out激活函数
