# 物体检测

在这里总结物体检测算法中一些重要的地方，比如特征表达、region proposal、背景信息的利用、训练策略等。
- [论文地址](https://arxiv.org/pdf/1809.02165.pdf)

## 特征表达

  特征提取的网络的重要性不言而喻，提出特征的好坏直接影响着检测任务的效果。
  1. 分类网络：
  ![](/pic/detect_1.png)
  
  2. 特征的再加工：
  
  一般而言直接用上面网络中的最后一层的特征即可，但是物体的大小是不同的，所以我们要利用不同scale的特征。
  一般而言越后面的特征拥有越大的感受野和更强的语义信息，对于一些变化也就越鲁棒但是分辨率就低一些细节的几何信息就丢失了。
  直觉上来说，我们需要根据物体的大小来使用特征，对于小物体我们可能要用底层的特征，因为它们具有更高的分辨率和更细节的几何信息。
  
  这里有几种方式来利用不同层的特征：
  
    - 单纯的用不同层的特征来进行检测。
    - 融合不同层的特征形成单一的特征层。
    - 融合不同层的特征，形成多层融合特征。
    
  如SSD就是第一种方式，FPN、RefineDet就是第三种方式。下图中是一些有代表性的特征融合的方法：
  
  [RefineDet cvpr2018](http://openaccess.thecvf.com/content_cvpr_2018/papers/Zhang_Single-Shot_Refinement_Neural_CVPR_2018_paper.pdf)
  <span id="jump">Hello World</span>
  
  ![](/pic/detect_2.png)
  
## 背景信息的建模：
  背景信息一般可分为全局的和局部的信息，全局信息可以提供一些图片级别、场景级别的信息，比如在室内的情况下出现船的概率会很低，在海滩的情况下出现船的概率就会提高。局部可以提供周围物体的关系信息，比如人和人站在一起，人在开车等等。一些方法如下表：
  ![](/pic/detect_3.png)
  
  
  局部背景信息的整合更多的是扩大检测框来提取更多的周围信息：
  
  [Relation Networks for Object Detection, ORN cvpr2018](http://openaccess.thecvf.com/content_cvpr_2018/papers/Hu_Relation_Networks_for_CVPR_2018_paper.pdf)
  
  [CoupleNet iccv2017](http://openaccess.thecvf.com/content_ICCV_2017/papers/Zhu_CoupleNet_Coupling_Global_ICCV_2017_paper.pdf)
  
  ![](/pic/detect_4.png)
  
## data argumentation：
  这里的数据增广主要针对解决物体scale的问题，下表为几种针对性的方法：
  
  [SNIP](http://openaccess.thecvf.com/content_cvpr_2018/papers/Singh_An_Analysis_of_CVPR_2018_paper.pdf)
  
  [SNIPER](http://papers.nips.cc/paper/8143-sniper-efficient-multi-scale-training.pdf)
  
  ![](/pic/detect_5.png)
  

[XXXX](#jump)
