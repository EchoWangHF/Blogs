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
如下图所示，从指令1到指令7,一共存在5个虚拟寄存器，每个虚拟寄存器的生命周期入下右图所示，现在存在两个物理寄存器r1，r2可以分配。

![basic_lra_1](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_lra_1.png)

对v1进行分配，此时active list为空，可以把r1物理寄存器分配给v1, 并将v1加入到active list当中。

对v2进行分配，因为v1的lifetime是从1到7的，所以v1依旧存在active当中，此时将物理寄存器r2分配给v2。

对v3进行分配，此时物理寄存器r1和r2都已经分配完，因此需要一个虚拟寄存器spill，因为v1的结束点最迟，因此选择v1进行spil，将v1从active list移除，将v3加入到active list当中。v3分配前后变化如下：

![basic_lra_2](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_lra_2.png)

对v4进行分配，此时v2生命周期结束，将v2从active当中移除，将v2的物理寄存器r2分配v4。

对v5进行分配，v3生命周期结束，将v3的物理寄存器r1分配给v5。
#### 缺陷：
由于basic算法的保守，要求所有的interval的生命周期都是连续的，不允许hole的存在，导致在分配的过程当中，v1不得不被spill到mem当中。分析指令后，发现v1在3被use后，在5被重新def了，所以中间存在一个hole，可以利用这个hole，先将v1的寄存器r1分配给v3,等v1需要的时候,v3也use完了，再分配给v1,进而避免spill，具体情况如下图所示：
![basic_lra_2](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_lra_3.png)

## 3 Optimized Interval Splitting In a Linear Scan Algorithm

#### basic step
![basic_step](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_step.png)

#### Block Order


