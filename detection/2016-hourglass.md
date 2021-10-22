## HourglassBackbone：Stacked Hourglass Networks for Human Pose Estimation

一个非常重要的backbone, 基于AE结构，可以堆叠。

Hourglass的核心驱动是需要捕获各个尺度的信息。局部的信息和全局的信息都可以被捕获，并且最后融合起来得到像素级别的输出。

### 单个Hourglass的处理pipeline

1. 在进入hourglass之前，先将图像得到对应的通道图。
2. 一系列的卷积和最大池化操作用来处理特征，下采样到非常低的分辨率尺寸，在每一个最大池化操作步骤，网络分支会应用更多的卷积卷积层。
3. 达到最低的分辨率之后，开始进行上采样完成top-down步骤。使用最近邻完成上采样步骤，然后像素级别的相加操作完成特征融合。
4. 整个hourglass的结构是对称的。每一层在最后都能找到与其对应的层。
5. 在完成hourglass特征提取之后，使用两个连续的1 * 1卷积来得到最终的网络输出，即一个heatmap集合，用来预测各个关节点。

### 单个Hourglass层的实现

初始卷积层 + residual层 [得到64 * 64的特征大小] -- > hourglass堆叠部分。所有的卷积操作使用的都是残差块来进行连接，实际上这里是不是可以简化？

最终结构的确定：采用3 * 3卷积和残差块结构。所有的hourglass结构权重不共享，并且所有的中间层损失所使用的gt都是一样的。

### 利用中间层监督的堆叠hourglass结构

论文里说中间层监督是非常重要的一个提升性能的途径，即将中间层得到的结果反馈到训练过程中去。

重复的bottom-top, top-down机制。能够有多次机会来重新评估整幅图像中的特征，从而在不同的级别特征得到多次处理。

### 损失部分

采用gaussian heatmap和MSE作为损失部分。

### 结论

在姿势估计方面取得了最好的结果。而且证明中间层的监督能够有效提升精度。