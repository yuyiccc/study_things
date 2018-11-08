## [S3FD: Single Shot Scale-invariant Face Detector](https://github.com/yujack333/study_things/blob/master/face-related%20paper/1708.SFD.pdf)
### 此文章为人脸检测算法文章：
- 网络结构：

  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD.png)
  
  
- 问题：(文章提出了4个问题)
  - 1. 小人脸需要更多的特征来检测，最低层的stride还是太大（8 or 16），文章用的最底层的feature map的stride=4。
  - 2. anchor size应该根据有效感受野来确定而不是理论感受野。因为有效感受野服从正态分布，且只是理论感受野的一部分。
  - 3. anchor size是离散的几个值，而真实的face size是连续的值。之前的匹配方式会导致小人脸和不在anchor size附近的人脸不能被anchor很好的匹配。
  - 4. 底层的feature map由于anchor太多导致大部分都是negative的。这样会导致有很多的假阳性（false positive）。
  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/S3FD1.png)

- 方法：
  - 1.文章的backbone用的是VGG，在VGG的基础上再加了额外的几层，具体可以看本文图一。
  
    解决第一个问题的方法就是用更低层的特征图，这样具有更高的分辨率来检测小人脸。
  - 
