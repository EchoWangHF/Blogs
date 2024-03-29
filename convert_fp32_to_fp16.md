# float32数据类型与float16数据类型如何互相转换

## 1. 基本概念介绍
`float32`数据类型就是我们常见的`32bit`的浮点数据类型， `float16`数据类型又称`half`数据类型，是`16bit`的浮点数据类型，是`NVIDIA`公司提出，被`IEEE754`规范，并在神经网络当中广泛使用的数据标准。`float16`的优点在于既具有基本的浮点数据类型功能，又具有低位宽的效果，在低带宽高算力的芯片(大多数TPU芯片都是低带宽高算力)能够降低IO量，提高算子或者网络整体的性能。但是`float16`的缺点也比较明显，就是精度不够，最大能表示的数值也才`65504`，计算过程当中丢失精度，进而导致神经网络不收敛。所以在很多低带宽高算力的TPU芯片上的神经网络算子都是以`float16`数据读入数据，在计算的过程当中转成`float32`，完成计算后再转成`float16`存储到片下ram空间。

`float32`的数据结构比较常见，符号位sign:`1bit`, 指数位exp:`8bit`, 尾数位frac:`24bit`, 偏置:`127`。`float16`的数据结构为：符号位sign:`1bit`, 指数位exp:`5bit`, 尾数位frac:`10bit`, 偏置:`15`。

`float16`的数值计算为：

