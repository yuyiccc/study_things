# Mask-rcnn解读
mask-rcnn是2017cvpr最佳论文，做的任务为object detection 和 instance segmentation。在这里写下自己读过之后的理解。
分为三部分：1.roi-Align，2.模型框架，3.结论。这篇论文最重要的地方在于对roi-pooling进行了改进，并基于faster-rcnn框架、
加上一支对roi的mask的预测来进行instance segmentation的任务。下面对三个部分一一讲解.

## Roi-Align vs RoiPooling
RoiPooling可以将不同大小的ROI的特征pooling到同样大小的特征。文章指出RoiPooling的两次量化会导致ROI区域和其提取出来的特征没有严格对准。
因此针对这两次量化对RoiPooling进行了改进，改进后的方法称为Roi-Align。

### RoiPooling
- 第一次量化：

得到ROI的坐标后（这个坐标实际上是在输入图像上的坐标），需要将坐标映射到特征图的空间上。在faster-rcnn中图片经过基础的卷积网络之后，大小变为原来的1/16.
映射方法:（x1,y1,x2,y2）-> (round(x1/16),round(y1/16),round(x2/16),round(y2/16)).其中round为小数点后四舍五入函数。效果图如下:

![](/pic/roipooling_1.png)

- 第二次量化：

第二次量化为：将roi的特征图划分bin的过程，这个量化与第一次的类似，也是需要round到一个整数。效果图如下：
![](/pic/roipooling_2.jpg)

这两次量化都有round过程，这导致了ROI区域和经过RoiPooling过后得到的特征图没有对准。

### Roi-Align
Roi-Align 方法则没有这两次量化过程。解决第一次量化的方法为不取round，（x1,y1,x2,y2）-> (x1/16,y1/16,x2/16,y2/16)。
解决第二个量化的方法为：先划分bin，再通过双线性差值的方式在每个bin中采样4个值，最后取这4个值max或者avg，也可以只采样一个值。
示意图如下：
![](/pic/roi_align.png)

由于没有进行round，所以采样点的坐标不为整数，例如计算（3.25,4.36）处的值。下面介绍如何通过双线性差值来计算。
![](/pic/bilinear_interpolation.png)

## 模型框架
