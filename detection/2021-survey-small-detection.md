## 小目标领域检测综述

#### 常见的组件

1. Region Proposals
2. Anchors
3. Object Classification
4. Bounding Box Regression
5. NMS

### OD方法

#### 基于锚框的方法

1. 图像 --> RoIs --> bbox --> 分类预测

#### 无锚框方法

==> ？？？无锚框方法会有相对比较大的优势

### 小目标检测

#### 问题

1. 各个层之间的特征是独立的，并且每特征层内部蕴含的信息是不充分的。对于小目标而言。

   1. 多层特征融合。自底向上或者自顶向下的过程来融合。
   2. ae结构，skip结构。

2. 小目标的上下文信息很有限。

   解决方法是和上下文信息模块合作，常用的方法是使用对于局部上下文则采用增大卷积核大小，是最常用的一种，对于语义上下文则是采用抽取更深层次的特征。比如使用转置卷积网络等。

   1. 局部上下文信息。
   2. 语义上下文。
   3. 空间上下文。

3. 小目标的类别不均衡问题。

   1. 采样。
   2. 损失加权。

4. 小样本的正样本不足问题

   1. 多分枝检测
   2. 匹配策略，对于锚框采用合适的匹配方法。
   3. 增加小目标正样本的数量。

#### 通用OD情况下的小目标检测方法

##### 提升小目标的特征

特征融合，低分辨率和高分辨率的特征全融合在一起，每一层都进行检测得到输出。自顶向下，自底向上不同的路径。典型结构如fpn， 以及ae结构。

1. AE结构，参考U-Net以及Hourglass。
2. FPN的多层级特征融合。
3. 采用转置卷积或者是双线性上采样过程。

##### 融合小目标的上下文信息

**在进行分类的时候适当增加bbox的范围，扩大到原来的1.5倍左右**。额外的语义信息和检测结合在一起。多个部分的特征会融合在一起再输入到检测和分类网络当中。

使用skip connections连接不同的层以提升小目标的检测效率。

使用RNNs逐像素级别捕获全局信息，然后使用1x1卷积将所有的信息融合在一起。

##### 锚框机制下修正前后景类别的权重

通过数据采样机制和两步的回归方法来平衡前后景。或者给那些比较难分辨的负样本更多的权重。

1. 基于数据的。使用bootstrap方法来采样那些效果比较差的样本，主要根据他们的损失值来判断那些样本是比较难的。
2. 基于权重的。对于那些比较难的样本使用更大的损失权重。

##### 增加小样本的样本数

字面意思，对于那些混合的数据集，将其中比较小的样本挑选出来，对他们进行过采样。而对于那些一般的降低他们的样本数量所带来的影响。认为制作更多的小样本检测数据集。

主要包括了多尺度学习，尺度迁移，以及锚框的适应性匹配等。

多尺度：变换图像的尺寸，增加训练样本的数量。不同阶段的特征一起使用以使用不同大小的目标。

尺度转换：学习原始图像到各个尺度图像的转换。

使用ml寻求更适合小目标的锚框。

#### 有关人脸的增强 和 航拍小对象的检测

整体流程基本上都一样 只不过将对应的论文又拿出来整理说了一遍而已。

##### 人脸部分

1. 人脸特征图的增强
   1. skip连接低级特征和高级特征
   2. 图像金字塔，将各种给尺度的特征融合后再输入到检测网络中。
   3. 融合后的特征在和原始特征相加，作为最终的检测特征。
   4. 不同的上采样方式，skip相连的位置和融合的策略等。
2. 增加语义信息
   1. 需要加入合适的上下文信息，太多或者太少都会影响最后的结果。
   2. 使用更大的卷积核而不仅仅是3 * 3.
   3. 使用实例分割网络来捕获更多的信息。
3. 修正前后景之间的类别平衡。
   1. 对于基于锚框的机制而言，会存在严重的正负样本不均衡问题。需要调整正负样本之间的比例。
      1. 将最大输出的背景标签，预测背景标签并且选择最大的一个作为最终得分。
      2. 使用一个两阶在低层去过滤那些假阳性的bbox, 从而达到平衡正负样本并提升分类结果的目的。
   2. 使用负样本挖掘来是的负样本和正样本之间的比例大约在3：1， 基于锚框的方法都集中于寻求一种采样机制使得正负样本之间的采样是平衡的。
4. 增加比较小的人脸的样本数目
   1. 小目标样本数很少的情况下自身的召回率就会很低。设计更合适的锚框用于匹配更多的人脸。考虑到人脸的尺寸和锚框的步长。另外可以对人脸部分进行偏移等，从而获得更高的IoU得分。
   2. 调整iou算法，使得较小的bbox也能获得比较高的iou.
5. 多尺度训练
   1. resize图像，生成一系列不同大小的对象，这样小的目标就会被方法，从而被检测到。??? resize是怎么做到的 ???
   2. 多个不同的卷积分支，分别检测不同尺寸的face. 小的目标通过conv4和conv5联合起来检测，中等大小的目标通过conv5检测，大的目标通过conv5最大池化之后再检测。
   3. 使用图像金字塔。分别适应一张图中大小不一的对象检测。（这里可以分成两组解耦头，一组用于球员，然后另外一组用于足球).

#### 航拍图像部分

航拍图像中有自己独特的特性，比如多尺度和多旋转角度等问题。旋转不变性的模型用来学习遥感图像中的小对象旋转问题。

1. 小目标的旋转问题。遥感图像中特有的旋转问题。
2. 小目标的上下文信息添加。同样利用多层特征融合来做。使用空洞卷积。
3. 修正前后景的类别分布。基于IoU进行检测来降低损失。提出基于IoU权重的损失函数来学习正样本的IoU信息，最后，提出了一种长宽比和尺寸限制的nms机制来提升检测性能。
4. 增加小目标检测的样本数量。

#### 用于小目标检测的实例分割方法

1. 核心结构：U-NET以及FPNs, 使用注意力机制等。

### 深度学习类方法表现的评估

1. Faster-RCNN Yolov3 SSD在不同数据集上的表现

### 小目标检测的发展方向

无锚框机制。









