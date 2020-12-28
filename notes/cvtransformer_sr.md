## Transformer in SR

### 简介

使用LR和Ref Images作为key和value对训练transformer，使其能够学习比较好的特征恢复能力。需要注意的是，SR一般有两种Paradigm，即单幅图像的SISR和基于参考的RefSR. 

传统的SISR通常会造成比较模糊的结果，原因是高分辨率纹理在降采样过程中被严重损坏了，不但很难恢复，而且往往是二义性的。

近期RefSR被广泛关注，即从一个参考图像中搜索并选择合适的HR纹理来生成视觉上比较令人满意的结果。一般的SOTA方法会存在视角变换无法搜索出正确纹理。

### 核心思想

利用transformer学习比较好的hard-attention和soft-attention. 主要有4部分组成，第一部分包括了特征抽取器，主要从LR图像和Ref中抽取特征。然后学习一个相关嵌入模块，即transformer模块，将LR中的特征作为key, 将Ref中的图像作为key, 从而获得硬注意力和软注意力的映射。最后分别使用硬注意力模块看和软注意力模块和一个自注意力模块将从Ref图像中搜索、转化和融合HR特征到LR图像中。

最后还学到了一个跨尺度的特征整合模块，从不同的尺度中学习到更富表现力的特征。 

### 方法模型

#### 可学习的纹理抽取模块

对于经过上采样之后的LR图像，Ref先降低后升采样图像，以及原始Ref图像，分别通过一组共享的卷积和池化抽取特征，分别得到Query, Key和Value三个特征映射图，Query和Key通过self-attention机制得到注意力权重，根据注意力权重分别得到hard注意力权重和soft注意力权重，其中hard权重是自相关后得分对应最大的那个位置索引，soft权重是对应的值。

#### 相关性嵌入模块

相关性嵌入主要是通过Q和K学习LR和Ref图像之间的相关性。将得到的Q和K拆分为多干个patches, 对于每一个qi以及kj, 计算二者的相关性，从而得到hard attention map 和soft attention map。

其中，hard attention map即为对应的最大的那个kj, 因此，通过这种方式从Ref对应的V中选择对应位置的特征，得到初级参考特征transferred HR Texture features。

soft attention map即为对应的那些最大的权重值。 首先将刚才得到的特征与LR特征融合在一起，然后进行元素点乘，最后再加回去。计算过程可以看作：
$$
F_{out} = F + Conv(Concat(F,T))·S \tag3
$$
就是一个点乘融合过程。

#### 跨尺度特征集成

利用采样将处理不同尺度的transformers堆叠起来，处理不同的分辨率。略。

#### 损失设计

重建损失 + 质量感知损失 + 对抗损失。

### 训练细节

使用Adam优化器，β1 = 0.9，β2 = 0.999，eps = 1e-8. lr = 1e-4, 采用warup机制，即刚开始的2 epoch仅仅优化重建损失，之后再开始优化所有的损失。

