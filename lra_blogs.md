# Linear Scan Register Allocation

## 1 Register Allocation

#### 寄存器分配的基本任务：
使用物理寄存器(physical register)替代虚拟寄存器(virtual register)。

#### 寄存器分配的目标：
尽可能地缓解内存带宽和处理器之间的性能冲突。

#### 寄存器分配的现实问题：
寄存器是有限的，不可能所以的值都存放在寄存器中，肯定会有值被溢出(spill)到栈(stack)上面的，等后面需要再使用的时候，再从stack上面load到register当中。

#### 寄存器分配算法的目的：
尽可能地减少register spill的发生。

#### 寄存器分配常见算法：
(1) 图着色算法 (Graph Coloring Algorithm) : 性能更好

(2) 线性扫描算法 (Linear Scan Algorithm)：速度更快

