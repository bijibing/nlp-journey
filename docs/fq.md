# 常见问题

> 整理、收集自网上资源

## 1. 使用cuda出现问题的解决方式

```bash
sudo ldconfig /usr/local/cuda-9.0/lib64
sudo ln -sf /usr/local/cuda-9.0/lib64/libcudnn.so.7.0.5 /usr/local/cuda-9.0/lib64/libcudnn.so.7
```

## 2. 找不到自己写的模块

**在代码入口处加入一下代码：**

```python
import sys
sys.path.append('/project_path/module')
```

## 3. CNN的feature map计算方式

![cnn](../images/cnn_feature_map.png)

## 4. RNN cell如何计算隐藏状态？

![rnn_cell](../images/rnn_cell.gif)

## 5. LSTM中各种门的作用？

LSTM 的核心概念在于**细胞状态**以及“**门**”结构。细胞状态相当于信息传输的路径，让信息能在序列连中传递下去。可以将其看作网络的“记忆”。理论上讲，细胞状态能够将序列处理过程中的相关信息一直传递下去。

LSTM 有三种类型的门结构：遗忘门、输入门和输出门。

**遗忘门**

遗忘门用于决定应丢弃或保留哪些信息。来自前一个隐藏状态的信息和当前输入的信息同时传递到 sigmoid 函数中去，输出值介于 0 和 1 之间，越接近 0 意味着越应该丢弃，越接近 1 意味着越应该保留。

![forget_gate](../images/lstm/forget_gate.gif)

**输入门**

输入门用于更新细胞状态。首先将前一层隐藏状态的信息和当前输入的信息传递到 sigmoid 函数中去。将值调整到 0~1 之间来决定要更新哪些信息。0 表示不重要，1 表示重要。

其次还要将前一层隐藏状态的信息和当前输入的信息传递到 tanh 函数中去，创造一个新的侯选值向量。最后将 sigmoid 的输出值与 tanh 的输出值相乘，sigmoid 的输出值将决定 tanh 的输出值中哪些信息是重要且需要保留下来的。

![input_gate](../images/lstm/input_gate.gif)

**细胞状态**

下一步，就是计算细胞状态。首先前一层的细胞状态与遗忘向量逐点相乘。如果它乘以接近 0 的值，意味着在新的细胞状态中，这些信息是需要丢弃掉的。然后再将该值与输入门的输出值逐点相加，将神经网络发现的新信息更新到细胞状态中去。至此，就得到了更新后的细胞状态。

![cell_state](../images/lstm/cell_state.gif)

**输出门**

输出门用来确定下一个隐藏状态的值。首先，我们将前一个隐藏状态和当前输入传递到 sigmoid 函数中，然后将新得到的细胞状态传递给 tanh 函数。

最后将 tanh 的输出与 sigmoid 的输出相乘，以确定隐藏状态应携带的信息。再将隐藏状态作为当前细胞的输出，把新的细胞状态和新的隐藏状态传递到下一个时间步长中去。

![output_gate](../images/lstm/output_gate.gif)

## 6. 如何解决样本不均衡的问题

**1. 产生新数据型：过采样小样本(SMOTE)，欠采样大样本**

**2. 对原数据的权值进行改变**

**3. 通过组合集成方法解决**

**4. 通过特征选择**

## 7. Dropout为什么能解决过拟合问题？

过拟合可以通过阻止某些特征的**协同作用**来缓解。在每次训练的时候，每个神经元有百分之50的几率被移除，可以让一个神经元的出现尽量不依赖于另外一个神经元。另外，可以把dropout理解为**模型平均**。

如果采用dropout，训练时间延长。dropout在数据量比较小的时候，不建议使用，效果并没有特别好，dropout的值一般来说取值为0.5。

## 8. 梯度消失和梯度爆炸如何产生？如何解决？

梯度消失的根源：深度神经网络和反向传播。

梯度消失经常出现，一是在深层网络中，二是采用了不合适的损失函数，比如sigmoid（sigmoid梯度不大于0.25，所以很容易发生梯度消失，tanh类似。）。

梯度爆炸一般出现在深层网络和权值初始化值太大的情况下。

解决办法：

* 预训练加微调：每次训练一层隐节点，在预训练完成后，再对整个网络进行“微调”。
* 梯度剪切、正则：设置一个梯度剪切阈值，更新梯度的时候，如果梯度超过这个阈值，就将其强制限制在这个范围之内。通过正则化项，可以部分限制梯度爆炸的发生。
* relu、leakrelu、elu等激活函数：如果激活函数的导数为1，那么就不存在梯度消失爆炸的问题了。
* batchnorm：批规范化，通过规范化操作将输出信号x规范化保证网络的稳定性。通过对每一层的输出规范为均值和方差一致的方法，消除了w ww带来的放大缩小的影响，进而解决梯度消失和爆炸的问题。
* 残差结构：短路机制可以无损地传播梯度，而另外一项残差梯度则需要经过带有weights的层，梯度不是直接传递过来的。残差梯度不会那么巧全为-1，而且就算其比较小，有1的存在也不会导致梯度消失。
* LSTM：LSTM通过它内部的“门”可以接下来更新的时候“记住”前几次训练的”残留记忆“。

> 这篇博文，[`梯度消失和梯度爆炸`](https://blog.csdn.net/qq_25737169/article/details/78847691) 讲的比较透彻。

## 9. 随机梯度下降的随机是啥意思？

批量梯度下降的时候，每次更新w的迭代，要对所有的样本进行计算，样本非常大的情况下，计算量太大。因此实用的办法是随机梯度下降SGD, 由于样本的噪音和随机性，每次更新w并不一定总是按照减少E的方向，但总体上还是沿着减少E的方向前进的。

这里的随机是w的更新有一定的方向随机性，并不是随机抽取样本。对于非凸函数来说，存在许多局部最小值。随机性有助于我们逃离某些很糟糕的局部最小值，从而获得一个更好的模型。

## 10. Dropout, DropConnect,变分Dropout,循环Dropout,Zoneout

Dropout暂时从网络中移除神经网络中的单元;

DropConnect通过随机丢弃权重而不是激活来扩展Dropout;

变分Dropout通过重复“输入，输出和循环层的每个时间步长相同的dropout掩码（在每个时间步骤丢弃相同的网络单元）;

循环Dropout使用网络的隐藏状态作为计算门值和小区更新以及使用dropout的子网络的输入来正则化子网络;

Zoneout不是将某些单位的激活设置为0，而是随机替换某些单位的激活与他们从前一个时间步的激活。

> 来自 [Dropout在LSTM / GRU递归神经网络中的应用详解](http://www.aboutyun.com/thread-25279-1-1.html)

## 11. Batch Normalization的优势

论文中将Batch Normalization的作用说得突破天际，好似一下解决了所有问题，下面就来一一列举一下：

　　(1) 可以使用更高的学习率。如果每层的scale不一致，实际上每层需要的学习率是不一样的，同一层不同维度的scale往往也需要不同大小的学习率，通常需要使用最小的那个学习率才能保证损失函数有效下降，Batch Normalization将每层、每维的scale保持一致，那么我们就可以直接使用较高的学习率进行优化。

　　(2) 移除或使用较低的dropout。 dropout是常用的防止overfitting的方法，而导致overfit的位置往往在数据边界处，如果初始化权重就已经落在数据内部，overfit现象就可以得到一定的缓解。论文中最后的模型分别使用10%、5%和0%的dropout训练模型，与之前的40%-50%相比，可以大大提高训练速度。

　　(3) 降低L2权重衰减系数。 还是一样的问题，边界处的局部最优往往有几维的权重（斜率）较大，使用L2衰减可以缓解这一问题，现在用了Batch Normalization，就可以把这个值降低了，论文中降低为原来的5倍。

　　(4) 取消Local Response Normalization层。 由于使用了一种Normalization，再使用LRN就显得没那么必要了。而且LRN实际上也没那么work。

　　(5) 减少图像扭曲的使用。 由于现在训练epoch数降低，所以要对输入数据少做一些扭曲，让神经网络多看看真实的数据。

## 12. 如何评价embedding的好坏？

其实，到目前为止，都没有很好的评价embedding好坏的方法，只能采用一些比较合理的方式，去抽样查看。

1. Relatedness：看看空间距离近的词，跟人的直觉是否一致
2. Analogy：著名 A - B = C - D 词汇类比任务
3. Categorization：看词在每个分类中的概率
4. 聚类算法（可视化）： kmeans 聚类，查看聚类分布效果 。若向量维度偏高，则对向量进行降维，并可视化。

## 13. 画一下lstm的原理图

![lstm](../images/lstm/lstm_full.png)

## 14. 为什么要加入Attention

输入序列非常长时，模型难以学到合理的向量表示；

序列输入时，随着序列的不断增长，原始根据时间步的方式的表现越来越差，这是由于原始的时间步模型设计的结构有缺陷，即所有的**上下文输入信息都被限制到固定长度**，整个模型的能力都同样受到限制，暂且把这种原始的模型称为简单的编解码器模型；

编解码器的结构无法解释，也就导致了其无法设计。

## 15、如何防止过拟合？

* 最好的降低过度拟合的方式之一就是增加训练样本的量。
* 降低网络复杂度。
* 规范化：权重衰减(weight decay)或者 L2 规范化。λ 越小，就偏向于最小化原始代价函数，反之，倾向于小的权重。
* L1 规范化：L1 规范化倾向于聚集网络的权重在 相对少量的高重要度连接上，而其他权重就会被驱使向 0 接近。
* 弃权：弃权过程就如同大量不同网络的效果的平均那样。
* 人为增加训练样本。

在科学中，一种观点是我们除非不得已应该追随更简单的解释。
