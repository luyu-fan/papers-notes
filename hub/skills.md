## 调参实录

### 训练的一般步骤

#### 搭建网络结构

#### 确定参数初始化方式

在pytorch中，卷积层中默认使用的初始化方式为kaiming_uniform, 针对的是ReLU激活函数设置，具备比较好的表现。线性层中同样使用的是kaiming_uniform方式。

```python
# 初始化方法
init.kaiming_uniform_(self.weight, a=math.sqrt(5))
if self.bias is not None:
    fan_in, _ = init._calculate_fan_in_and_fan_out(self.weight)
    bound = 1 / math.sqrt(fan_in)
    init.uniform_(self.bias, -bound, bound)
```

如果是自定义的网络类（非容器类）则可以直接重写这个类的函数，对权重和偏置直接使用某一种初始化函数赋值。

```python
def reset_parameters(self) -> None:
        init.kaiming_uniform_(self.weight, a=math.sqrt(5))
        if self.bias is not None:
            fan_in, _ = init._calculate_fan_in_and_fan_out(self.weight)
            bound = 1 / math.sqrt(fan_in)
            init.uniform_(self.bias, -bound, bound)
```

如果需要自定义某些层的参数初始化方式，则需要使用nn.init模块进行自定义，然后可以对某些模块进行apply将其应用到上面去。

代码如下：
```python
def weights_init(m):
    if isinstance(m,nn.Linear):
        nn.init.xavier_normal_(m.weight)
        if m.bias is not None:
            nn.init.constant_(m.bias,0)
    elif isinstance(m,[nn.Conv1d,nn.Conv2d,nn.Conv3d]):
        nn.init.kaiming_uniform_(m.weight)
        if m.bias is not None:
            nn.init.constant_(m.bias,0)
    else:
        ...
net = Net()
# apply函数会递归地搜索网络内的所有module并把参数表示的函数应用到所有的module上。
net.apply(weights_init) 
```

一般情况下，如果和ReLU函数结合使用，则最好使用kaiming_uniform方法，如果需要结合具体的任务则可以重写具体类的方法（最基本的构成类）或者是将整个初始化方法利用apply放到整个模型上去。

#### 调整合适学习策略

需要和具体的优化函数结合使用，需要重点考虑的是需不需要使用warmup机制。



