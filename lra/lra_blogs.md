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

## 2 Basic Linear Scan Algorithm
#### 基本内容：
(1) 所有的指令首先被线性排序，条件判断和循环等控制流指令都会被隐藏起来，后续再通过数据流分析的方法去处理控制流指令的影响。

(2) 使用interval的数据结构去表示每一个virtual register， interval记录了virtual register的生命周期，即从 first definition 到 last use。

(3) 每一个interval的lifetime是连续的，从first definition 到 last use，不允许中断，不允许hole的存在。(弊端)

(4) 算法在运行当中，会维护一个active的list，当中存放已经分配register，但是没有结束的interval。
#### 举例说明：
![basic_lra_1](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_lra_1.png)


