---
layout: wiki
title: pytorch-网络模块
cate1: pytorch
---

## torch.nn.embedding

词嵌入网络，就相当于我们在给计算机制造出一本字典的过程，计算机可以通过这个字典来间接识别文字，词嵌入向量的意思是：词在神经网络中的向量表示。

参数：

- num_embeddings：词嵌入字典的大小，即一个字典里要有多少个词
- embedding_dim：每个词嵌入向量的大小

例子：

```python
import torch
import torch.nn as nn

embedding = nn.Embedding(10, 3)
input = torch.LongTensor([[1,2,4,5],[4,3,2,9]])
e = embedding(input)
print(e)
```

输出的结果：

```
tensor([[[-0.0251, -1.6902,  0.7172],
         [-0.6431,  0.0748,  0.6969],
         [ 1.4970,  1.3448, -0.9685],
         [-0.3677, -2.7265, -0.1685]],

        [[ 1.4970,  1.3448, -0.9685],
         [ 0.4362, -0.4004,  0.9400],
         [-0.6431,  0.0748,  0.6969],
         [ 0.9124, -2.3616,  1.1151]]])
```

理解代码：

- 首先，`embedding = nn.Embedding(10, 3)` 定义了一个 embedding 模块，包括一个长度为 10 的张量，每个张量的大小是 3。
- 其次，`input = torch.LongTensor([[1,2,4,5],[4,3,2,9]])` 即输入一个我们需要 embedding 的变量，输入的每个值最终**嵌入**到张量空间中

再举个例子，我们看看 embedding 的 weight：

```
tentor([[-0.4794, -1.0073,  0.6200],
		[-1.0556, -0.2404, -0.4578],
        [ 1.3328,  2.5743, -0.7375],
        [ 0.4732,  0.2627, -0.9978],
        [-0.0584, -0.6458,  0.8236],
        [ 0.3959,  0.1437,  0.2200],
        [-0.4259, -0.1086, -0.7421],
        [ 0.4835, -2.0671,  0.7834],
        [ 0.0622, -0.0935,  1.0996],
        [ 0.6687, -1.4764,  1.5658]]], requires_grad=True)
```

我们发现 `embedding.weight` 是一个 [10, 3] 的向量。如果 `index=1`，那我们取 [-1.0556, -0.2404, -0.4578]，如果 `index=2`，那我们取 [ 1.3328,  2.5743, -0.7375]，在经过 embedding 后，原本 [2,4] 的维度，就变换为了 [2,4,3]，其实就是 [2,4] 中的每个值作为索引去 `nn.embedding` 中取对应的权重。

## RMSNorm

在每个向量矩阵计算之前，需要对输入向量进行 normalization，之前使用的是 layer norm，现在使用 RMSNorm，两者的区别在于：

- layer norm：减去样本的均值，除以样本的方差，使整体样本不要太分散

$$
y = \frac{x-E(x)}{\sqrt{Var(x) + \epsilon}} * \gamma + \beta
$$

- RMS(root mean square) norm：省去了减去均值的操作，去中心化操作

$$
\overline{a}_i = \frac{a_i}{RMS(a)}g_i
$$

其中

$$
RMS(a) = \sqrt{\frac{1}{n}\sum_{i=1}^{n}{a_i^2}}
$$

pytorch 代码实现：

```python
class RMSNorm(nn.Module):
    """
        RMSNorm（Root Mean Square Layer Normalization）: 均方根层归一化，是一种层归一化方法

        Args:
            size: 输入张量维度大小
            dim: 沿指定维度计算 RMSNorm
            eps: 用于避免除以 0

        Note:
            此实现基于 https://github.com/bzhangGo/rmsnorm/blob/master/rmsnorm_torch.py
            遵循 https://github.com/bzhangGo/rmsnorm/blob/master/LICENSE
    """

    def __init__(self, size: int, dim: int = -1, eps: float = 1e-5) -> None:
        super().__init__()
        self.scale = nn.Parameter(torch.ones(size))  # 可学习缩放参数
        self.eps = eps  # 防止除 0 的小数值
        self.dim = dim  # 用于计算 RMSNorm 的维度

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """
            执行 RMSNorm 前向传播

            Args:
                x: 输入张量

            Returns:
                torch.Tensor: 经过 RMSNorm 处理后的张量

            Note:
                原始的 RMSNorm 论文实现与此处的计算方法略有不同

        """
        norm_x = torch.mean(x * x, dim=self.dim, keepdim=True)
        x_normed = x * torch.rsqrt(norm_x + self.eps)
        return self.scale * x_normed
```
