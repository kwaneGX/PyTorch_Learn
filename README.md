# PyTorch：实现RNN中的批量梯度下降（BGD）
之前做过使用RNN进行句子分类的实验，由于当时没有足够的时间，只是草草地用PyTorch写了一个简单的LSTM模型，当时的程序没有实现批量梯度下降，只能进行随机梯度下降，训练速度很慢。
为此，这次我尝试使用PyTorch实现批量梯度下降。

## 问题描述
我们往往使用RNN对序列进行处理，序列较单词而言，其长度是可变的，这就为编程带来了困难。
为了能加快模型的训练效率，常常需要同时将多个序列输入模型进行梯度下降更新参数，这也就是批量梯度下降。
输入序列会被转化为Tensor输入网络，BGD的Tensor形状为:
\[batch size, time step, embedding dim\]
其中，
batch size是每次输入模型序列数目，
time step是输入序列的指定长度，
embedding dim是词向量的维度。

当用户指定了time step的大小后，我们需要对序列进行处理，长度不足的序列需要补长，长度超过的序列需要截断。
同时，为了在补长和截断完成后，仍然能找到序列的真正结尾处，我们需要额外存储一个mask，用于记录每个序列的真正结尾。

大致过程如下：
原始数据:

\['I am test sentence one', 'You are the one he like', 'I like eating'\]

字典映射后数据:

\[array(\[0, 1, 2, 3, 4\]), array(\[5, 6, 7, 4, 8, 9\]), array(\[ 0,  9, 10\])\]

处理后数据:

\[\[ 0  1  2  3  4\]

 \[ 5  6  7  4  8\]

 \[ 0  9 10  0  0\]\]


mask:

\[\[0 0 0 0 1\]

 \[0 0 0 0 1\]

 \[0 0 1 0 0\]\]


 然后可以将处理后的数据输入网络，得到网络输出后，用mask去获取真正句子结尾对应的隐藏状态。