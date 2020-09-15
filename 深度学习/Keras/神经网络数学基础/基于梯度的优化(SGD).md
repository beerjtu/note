## 神经网络的引擎：基于梯度的优化
### 梯度：是张量运算的导数
> 假设有一个输入向量 x 、一个矩阵 W 、一个目标 y 和一个损失函数 loss 。你可以用 W 来计算预测值 y_pred ，然后计算损失，或者说预测值 y_pred 和目标 y 之间的距离。

y_pred = dot(W, x)

loss_value = loss(y_pred, y)

>如果输入数据 x 和 y 保持不变，那么这可以看作将 W 映射到损失值的函数。

loss_value = f(W)
### 随机梯度下降
* 使用 2.4 节开头总结的四步算法：基于当前在随机数据批量上的损失，一点一点地对参数进行调节。由于处理的是一个可微函数，你可以计算出它的梯度，从而有效地实现第四步。沿着梯度的反方向更新权重，损失每次都会变小一点。
    1. 抽取训练样本 x 和对应目标 y 组成的数据批量。
    2. 在 x 上运行网络，得到预测值 y_pred 。
    3. 计算网络在这批数据上的损失，用于衡量 y_pred 和 y 之间的距离。
    4. 计算损失相对于网络参数的梯度［一次反向传播（backward pass）］。
    5. 将参数沿着梯度的反方向移动一点，比如 W -= step * gradient ，从而使这批数据上的损失减小一点。
* 刚刚描述的方法叫作小批量随机梯度下降（mini-batch stochastic gradient descent，又称为小批量 SGD）。术语随机（stochastic）是指每批数据都是随机抽取的（stochastic 是 random在科学上的同义词）。
* 步长(step)，也叫学习率
* 动量：如果使用小学习率的 SGD 进行优化，那么优化过程可能会陷入局部极小点，导致无法找到全局最小点。
* 有一种有用的思维图像，就是将优化过程想象成一个小球从损失函数曲线上滚下来。如果小球的动量足够大，那么它不会卡在峡谷里，最终会到达全局最小点。动量方法的实现过程是每一步都移动小球，不仅要考虑当前的斜率值（当前的加速度），还要考虑当前的速度（来自于之前的加速度）。这在实践中的是指，更新参数 w 不仅要考虑当前的梯度值，还要考虑上一次的参数更新，其简单实现如下所示。
``` python
past_velocity = 0.
# 不变的动量因子
momentum = 0.1
# 优化循环
while loss > 0.01:
    w, loss, gradient = get_current_parameters()
    velocity = past_velocity * momentum - learning_rate * gradient
    w = w + momentum * velocity - learning_rate * gradient
    past_velocity = velocity
    update_parameter(w)
```
### 链式求导：反向传播算法
* 下面这个网络 f 包含 3 个张量运算 a 、 b 和 c ，还有 3 个权重矩阵 W1 、 W2 和 W3： f(W1, W2, W3) = a(W1, b(W2, c(W3)))
* 根据微积分的知识，这种函数链可以利用下面这个恒等式进行求导，它称为链式法则（chainrule）： (f(g(x)))' = f'(g(x)) * g'(x) 。将链式法则应用于神经网络梯度值的计算，得到的算法叫作反向传播（backpropagation，有时也叫反式微分，reverse-mode differentiation）。反向传播从最终损失值开始，从最顶层反向作用至最底层，利用链式法则计算每个参数对损失值的贡献大小。
