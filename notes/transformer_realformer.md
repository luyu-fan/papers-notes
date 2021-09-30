# RealFormer

## 简介

由于LayerNorm在transformer中会有一定的问题，一般的Post-Norm形式性能和效果会比较好，但是不稳定，而且需要热重启机制，Pre-Norm机制会比较稳定但是效果比较差，而RealFormer则兼容了两者的优点。

## 方法及核心思想

创建一个最直接的传播原始注意力得分的，贯穿整个网络的路径。

### 传统的Transformer

传统的transformer无非采用两种结构，Post-Norm和Pre-Norm。

所谓的Post-Norm即是在每个子层后面添加一个LayerNorm，即为原始的transformer结构，在BERT和XLNet中，ALBERT都使用的是此类方法。

Pre-Norm则是指的是重新组织子层的顺序来形成一种完全“clean"的反向传播路径。主要结构为对于输入，分为两个分支，一个分支原封不动进行残差连接，另外一个分支则是先进行Norm规范化后然后再进行多头注意力机制，得到的输出再和刚才的原始输入进行残差相连然后进行输入下一个子层。这种结构在GPT-2中使用到。

值得注意的是，虽然Pre-Norm结构在ResNet b中表现的比Post要好，但是在自注意力机制中，似乎网络性能的偏好并不同于传统的CNN结构。

### RealFormer

核心做法是将前面层得到的原始注意力得分以残差连接的形式传递到后面层，就像ResNet当中那样做，即打通一条从头到尾的原始注意力得分（即矩阵乘法的相似度值）传播的通路。将两个相似性得分相加之后作为当前层新的原始得分再利用softmax产生线性相加的权重产生新的特征，同时，这个新的原始得分会传播到后面子层去。

Realformer中最大的特点是，通过定性分析，发现注意力会变得稀疏并且各个层之间相关性更高。

## 模型方法

RealNorm依然采用的是Post-Norm的形式，唯一不同之处就是增加了原始权重的skip connection. 并且传播到后面层中，其它并没有不同之处。唯一的不同之处：
$$
ResidualAttention(Q,K,V) = Softmax(\frac{QK^T}{\sqrt{d_k}} + Prev)V
$$
这个机制可以应用到其它任务当中去。

## 训练及实验

10% dropout，learning rate：1e-4, warm up然后线性衰减到0。

## 贡献

NLP上取得了最佳任务。

训练budget不高的情况下依然能取得比较好的结果。

能够使用比较大的学习率且能带来一定的好处，而一般的Post-Norm就会崩溃。

注意力会变得稀疏。









