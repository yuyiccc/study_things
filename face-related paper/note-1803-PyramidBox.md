## [PyramidBox: A Context-assisted Single Shot Face Detector](https://github.com/yujack333/study_things/blob/master/face-related%20paper/1803.PyramidBox.pdf)
## 此文章为人脸检测算法文章：
### 网络结构：

  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/PyramidBox_struct.png)
  
### 问题&方法：文章提出:1.小，模糊，有遮挡的人脸检测起来任然有很多问题。2.针对这个问题，作者提出背景特征对于这个问题有很大的帮助。所以提出3点。
  - 1. 基于FPN的方法可以有效利用high-level的特征，但是之前都是利用最高层的feature来融合。作者认为对于小物体而言最高层的特征反而不是最好的。
  原因有二：1.有遮挡且模糊的小人脸与清晰的完整的大人脸的纹理完全不同，2最高层的feature的感受野大，对于小人脸而言会引入噪声信息。所以这里作者不再用
  最高层的特征来向下融合，而是用中层的特征来向下融合。
  
  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/pyramid_boxes_lfpn.png)
  
  - 2. 有监督的利用背景特征可以更好的提高模糊，有遮挡的小人脸的检测。这一点从mask-rcnn中也可以看出。但这里没有额外的监督标签可以给出，于是作者
  给出了一个假设：人脸大一圈的区域为人头，人脸大两圈的区域为身体（通俗理解）。这样，在预测时，不仅要预测人脸，还要预测头和身体。只是头和身体的标签也是
  依据人脸的标签来给出的。
    - 1. 预测部分：Context-sensitive Predict Module (CPM)
    ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/pyramid_box_cmp.png)
    - 2.标签match方法
    ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/pyramid_box_match.png)
  
  - 3.Data-anchor-sampling
    ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/pyramid_box_augmentation.png)

### 结果&结论：
  - 1.结果：
  ![](https://github.com/yujack333/study_things/blob/master/face-related%20paper/pic/pyramid_box_result.png)
  - 2.结论：应验了上述假说
