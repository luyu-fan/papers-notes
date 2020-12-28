## Feature Pyramid Transformer

### 简介

空间语义内容能够随着感受野的增加而增加，然而非局部的空间交互信息并不是跨尺度的，并且也不能跨尺度捕获这种非局部信息。

考虑到某些实例对象的出现是有固定模式的，例如鼠标、电脑和显示屏是有着固定模式的，然而它们的尺度大小都不尽相同，所以应该在非局部的情况下捕获全局关联信息，简而言之，即捕获跨尺度、跨空间位置下的关系和依赖性。

### 核心想法

利用各种transformer完成同一个level下的自注意力、跨level下的自注意力，从而捕获称之为所谓non-local的interaction。

### 模型方法

首先从一个backbone中抽取出各个stage的特征，feature pyramid. 然后，分别建立下面三种transformer机制：

* self-transformer：捕获不同level中的对象之间的关联关系。使用了Mixture Softmaxes.
* grounding transformer：自顶向下用于分割。寻求高层“concepts”在低层中的印证。
* rendering transformer：通过组合低层像素特征的视觉特性来渲染高层的"概念"。使用全局特征的形式。

### 训练细节

学习率使用指数衰减，其中初始学习率为0.01，power为0.9，权重衰减4e-5, 动量为0.9，GPUs.