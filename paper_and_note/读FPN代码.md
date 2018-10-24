## FPN代码
![代码地址](https://github.com/yangxue0827/FPN_Tensorflow)


## 关于ResNet：
ResNet有两个版本：v1，v2。区别在于relu，bn，conv的顺序不同。
v1：
```python
def bottleneck(inputs, depth, depth_bottleneck, stride, rate=1,
               outputs_collections=None, scope=None):
  """Bottleneck residual unit variant with BN after convolutions.

  This is the original residual unit proposed in [1]. See Fig. 1(a) of [2] for
  its definition. Note that we use here the bottleneck variant which has an
  extra bottleneck layer.

  When putting together two consecutive ResNet blocks that use this unit, one
  should use stride = 2 in the last unit of the first block.

  Args:
    inputs: A tensor of size [batch, height, width, channels].
    depth: The depth of the ResNet unit output.
    depth_bottleneck: The depth of the bottleneck layers.
    stride: The ResNet unit's stride. Determines the amount of downsampling of
      the units output compared to its input.
    rate: An integer, rate for atrous convolution.
    outputs_collections: Collection to add the ResNet unit output.
    scope: Optional variable_scope.

  Returns:
    The ResNet unit's output.
  """
  with tf.variable_scope(scope, 'bottleneck_v1', [inputs]) as sc:
    depth_in = slim.utils.last_dimension(inputs.get_shape(), min_rank=4)
    ###
    分为三种情况，depth==depth_in 表示同一类unit（depth相同表示feature map size相同），
    如果stride==1则表示不是最后一个需要downsample的unit，则直接用输入作为shortcut。
    如果stride==2则表示为最后一个需要downsample的unit，则利用maxpooling（stride=2）来downsample
    如果depth!=depth_in则表示为第一个unit（前一个unit与此不是同一类unit），则用1*1的conv来使得depth_in==depth。
    ###
    if depth == depth_in:
      shortcut = resnet_utils.subsample(inputs, stride, 'shortcut')
    else:
      shortcut = slim.conv2d(inputs, depth, [1, 1], stride=stride,
                             activation_fn=None, scope='shortcut')

    residual = slim.conv2d(inputs, depth_bottleneck, [1, 1], stride=1,
                           scope='conv1')
    residual = resnet_utils.conv2d_same(residual, depth_bottleneck, 3, stride,
                                        rate=rate, scope='conv2')
    residual = slim.conv2d(residual, depth, [1, 1], stride=1,
                           activation_fn=None, scope='conv3')

    output = tf.nn.relu(shortcut + residual)

    return slim.utils.collect_named_outputs(outputs_collections,
                                            sc.original_name_scope,
                                            output)
```
- 代码的几点感悟：
- 1.代码应该具有高复用性，即同一功能的代码应该尽量的重用。
 - 在这里：用network_factory.py中的get_network_byname函数根据传入的网络名字来返回需要的网络。
 - 在调用resnet时，由于resnet有两种版本，所以将两个版本需要用到的共同的模块放在resnet_utils.py.
 - 而在resnet_v1.py中又有几个变种的网络（resnet50，resnet101，resnet152，resnet200），而这些变种的bottlenet的架构一样，
   整体的架构一样（resnet_v1），所以这两个函数是共用的。
 
