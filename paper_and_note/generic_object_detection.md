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
 
  [RefineDet_总结](#jump_1)
  
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
  

***
## 文章总结

<h2 id="jump_1">RefineDet 总结</h2>

### 整体框架

  这个研究主要是提出了一种介于‘一步’和‘两步’detector的一种新的检测器，目的是为了综合两种方法的优势，避免两种方法的短处。‘一步’方法，如yolo系列，SSD系列等的优点在于结构简单，速度快，缺点是精度不如‘两步’方法高。‘两步’方法，如faster-rcnn架构，优点是精度高尤其是定位，这是因为经过了两次的回归操作来调整框的位置，缺点是速度慢。作者结合这两种方法提出的架构图如下：

   ![](/pic/refinedet_1.png)

  其实可以把这个框架理解为两步‘一步’方法，即‘一步’方法做了两次。这个框架主要由三部分组成：Anchor Refinement Module（ARM）、Transfer Connection Block（TCB）、Object Detection Module（ODM）。其中ARM、ODM就是两次‘一步’方法,ARM中的分类是二分类即背景或者前景，ODM就是纯粹的SSD，而TCB可以理解为FPN，只不过向后传递的BLOCK的方法不同。下图就是TCB的结构：

  ![](/pic/refinedet_2.png)

### 细节部分
  ‘一步’方法精度低的一个主要原因是类别不均衡问题，比如负样本会domain loss，所以文中用到了几个方法来抑制这种情况的出现：

    - hard negative mining： 在匹配完之后，有很多的anchor的标签会使背景，那么这就会造成前景-背景的不平衡，所以在这里用到这种方法。
    具体的就是选择loss value大的 negative 样本使得negative：positive的比例不超过3:1。这个方法在ARM和ODM的loss中都会用到
    - 对于ODM而言还会将ARM预测的negative value超过阈值（0.99）的anchor排除掉，这是排除一些一定是背景的anchor。
  
  anchor的设计和匹配：这里用到的feature map的stride分别为8、16、32、64，而对应的anchor大小就是stride的4倍，有三个ratio分别是0.5、1、2。匹配方法是：1.与GT框最大iou的anchor标定为positive，2.与任意GT框的iou大于0.5的anchor标定为positive。第一条保证了每个GT框会有anchor与之匹配。
  
由于本文的loss和SSD很想，且matching method、hard negative mining方法与SSD一样，所以在这里展示SSD的这三部分代码：
  
  - matching method方法，代码的方法不完全对，它不能完全满足第一条，也就是说有的GT框可能没有anchor来匹配。
  ```python
      # 这里i是GT框的index，也就是对GT框继续循环
      # Jaccard score.先算所有anchor与一个GT框的IOU
      label = labels[i]
      bbox = bboxes[i]
      jaccard = jaccard_with_anchors(bbox)
      # GT-i框与某个anchor的IOU大于之前GT的IOU的话就更新这个anchor所对应的GT框
      # Mask: check threshold + scores + no annotations + num_classes.
      mask = tf.greater(jaccard, feat_scores)
      # mask = tf.logical_and(mask, tf.greater(jaccard, matching_threshold))
      mask = tf.logical_and(mask, feat_scores > -0.5)
      mask = tf.logical_and(mask, label < num_classes)
      imask = tf.cast(mask, tf.int64)
      fmask = tf.cast(mask, dtype)
      # Update values using mask.
      feat_labels = imask * label + (1 - imask) * feat_labels
      feat_scores = tf.where(mask, jaccard, feat_scores)

      feat_ymin = fmask * bbox[0] + (1 - fmask) * feat_ymin
      feat_xmin = fmask * bbox[1] + (1 - fmask) * feat_xmin
      feat_ymax = fmask * bbox[2] + (1 - fmask) * feat_ymax
      feat_xmax = fmask * bbox[3] + (1 - fmask) * feat_xmax
      
      # 这部分是对所有GT框进行循环
      i = 0
      [i, feat_labels, feat_scores,
      feat_ymin, feat_xmin,
      feat_ymax, feat_xmax] = tf.while_loop(condition, body,
                                            [i, feat_labels, feat_scores,
                                              feat_ymin, feat_xmin,
                                              feat_ymax, feat_xmax])
      # Transform to center / size. 转换
      feat_cy = (feat_ymax + feat_ymin) / 2.
      feat_cx = (feat_xmax + feat_xmin) / 2.
      feat_h = feat_ymax - feat_ymin
      feat_w = feat_xmax - feat_xmin
      # Encode features. 编码
      feat_cy = (feat_cy - yref) / href / prior_scaling[0]
      feat_cx = (feat_cx - xref) / wref / prior_scaling[1]
      feat_h = tf.log(feat_h / href) / prior_scaling[2]
      feat_w = tf.log(feat_w / wref) / prior_scaling[3]
      # Use SSD ordering: x / y / w / h instead of ours. anchor对应的坐标标签
      feat_localizations = tf.stack([feat_cx, feat_cy, feat_w, feat_h], axis=-1)
      # cls标签，loca标签，anchor与所有GT最大的IOU（shape与anchor相同）。
      return feat_labels, feat_localizations, feat_scores
      
      # Compute positive matching mask...
      # 这里利用上述IOU结果，就可得出匹配结果。当然这里有缺陷，匹配的第一条有时不能满足。
      pmask = gscores > match_threshold
      fpmask = tf.cast(pmask, dtype)
      n_positives = tf.reduce_sum(fpmask)     
  ```
  - hard negative mining 方法
  ```python
    # Hard negative mining...
    no_classes = tf.cast(pmask, tf.int32)
    predictions = slim.softmax(logits)
    nmask = tf.logical_and(tf.logical_not(pmask),
                           gscores > -0.5)
    fnmask = tf.cast(nmask, dtype)
    nvalues = tf.where(nmask,
                       predictions[:, 0],
                       1. - fnmask)
    nvalues_flat = tf.reshape(nvalues, [-1])
    # Number of negative entries to select.
    max_neg_entries = tf.cast(tf.reduce_sum(fnmask), tf.int32)
    n_neg = tf.cast(negative_ratio * n_positives, tf.int32) + batch_size
    n_neg = tf.minimum(n_neg, max_neg_entries)

    val, idxes = tf.nn.top_k(-nvalues_flat, k=n_neg)
    max_hard_pred = -val[-1] # negative prediction 的阈值
    # Final negative mask.
    #小于上述阈值就表明model认为不是negative但是它确实是negative，所以是hard negative
    nmask = tf.logical_and(nmask, nvalues < max_hard_pred) 
    fnmask = tf.cast(nmask, dtype)
  ```
  - loss方法
  ```python
    # Add cross-entropy loss.
    with tf.name_scope('cross_entropy_pos'):
        loss = tf.nn.sparse_softmax_cross_entropy_with_logits(logits=logits,
                                                              labels=gclasses)
        loss = tf.div(tf.reduce_sum(loss * fpmask), batch_size, name='value')
        tf.losses.add_loss(loss)

    with tf.name_scope('cross_entropy_neg'):
        loss = tf.nn.sparse_softmax_cross_entropy_with_logits(logits=logits,
                                                              labels=no_classes)
        loss = tf.div(tf.reduce_sum(loss * fnmask), batch_size, name='value')
        tf.losses.add_loss(loss)

    # Add localization loss: smooth L1, L2, ...
    with tf.name_scope('localization'):
        # Weights Tensor: positive mask + random negative.
        weights = tf.expand_dims(alpha * fpmask, axis=-1)
        loss = custom_layers.abs_smooth(localisations - glocalisations)
        loss = tf.div(tf.reduce_sum(loss * weights), batch_size, name='value')
        tf.losses.add_loss(loss)
  ```
