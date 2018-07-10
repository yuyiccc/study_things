## MobileNets

MobileNet是针对网络压缩提出来的，是希望将网络的参数和计算量都减少。在此基础上性能不会有太多的下降。
提出的方法是将传统的卷积操作进行分解。分解为两步：1.depthwise convolution(对feature map进行逐层的卷积) 
2.pointwise convolution(用N个1*1*M的卷积对第一步得到的feature map进行卷积得到N个feature map)

![](/pic/mobilenet.png)

下面来分析分解后的卷积操作在参数量和计算量上的减少情况：

- 参数量
  传统卷积的参数量为：![](/pic/m_p_0.gif) Dk为卷积核的大小，M为feature map的depth，N为有多少卷积核。
  
  分解后的第一步的参数量： ![](/pic/m_p_1.gif)
  
  分解后的第二步的参数量： ![](/pic/m_p_2.gif)
  

