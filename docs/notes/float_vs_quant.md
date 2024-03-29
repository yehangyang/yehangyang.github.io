# 浮点数 v.s. 定点数

## 浮点数更适用于训练

浮点数具备动态范围大(值域范围大)的特点. 动态范围大对于训练很有必要. 尤其是在模型训练的早期, 模型参数处于随机初始化状态, 导致模型 $loss$ 非常大, 很容易在反向传递(甚至是前向推理)时出现极大值. 例如在做 FP16 混合精度训练时, 有些层的数值容易出现超出 FP16 可以表达的最大值 65504; 如果此时这些层换成定点数(e.g., INT8)就更应该注意是否会超出范围.

不仅如此, 浮点数还具备靠近 0 的步长小, 远离 0 的步长大. 呈现的效果就是靠近 0 的数很密集, 远离 0 的数很稀疏. 换而言之, 浮点数可以表征细小的变化, 而在训练过程中, 因为梯度值通常都很小, 正好需要能够表征这些细小的变化, 这样训练能够非常平滑而稳定. 如果可表达数之间的步长很大, 就有可能因为梯度数值太小而被 "淹没".

## 数值计算稳定性

数值稳定性上定点数要优于浮点数. 浮点数在做加减法时, 其中有一步是对齐 $exponent$, 再进行 $mantissa$ 位的加减. 如果加减法只发生在两个数之间, 事件是确定的. 但是当发生在多个数进行累加/减时, 输出就会受计算顺序的影响(尽管数学上符合交换律).

具体讲, 以累加为例, 假设元素按照原顺序依次累加, 累加到一定数量就会出现 "超大数 + 超小数" 的现象. i.e., "超小数" 为了对齐 $exponent$ 位, 右移 $mantissa$ 位过多而变 0, 于是 "超小数" 就被 "超大数" 吞了, 变得毫无价值. 换个思路, 对所有元素按数量分组, 组内求和后, 对和再按数量分组, 再在组内求和, 直到所有元素都加到一起. 这样能在一定程度上避免 "超大数 + 超小数" 的现象出现.

反观定点数就没有这样的问题. e.g., 多个整数相加/减, 只要结果(包括中间结果)不会溢出, 不管怎么调换计算顺序, 结果也不会有变化.
