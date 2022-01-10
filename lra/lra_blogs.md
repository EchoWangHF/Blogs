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
#### 2.1 基本内容：
(1) 所有的指令首先被线性排序，条件判断和循环等控制流指令都会被隐藏起来，后续再通过数据流分析的方法去处理控制流指令的影响。

(2) 使用interval的数据结构去表示每一个virtual register， interval记录了virtual register的生命周期，即从 first definition 到 last use。

(3) 每一个interval的lifetime是连续的，从first definition 到 last use，不允许中断，不允许hole的存在。(弊端)

(4) 算法在运行当中，会维护一个active的list，当中存放已经分配register，但是没有结束的interval。
#### 2.2 举例说明：
如下图所示，从指令1到指令7,一共存在5个虚拟寄存器，每个虚拟寄存器的生命周期入下右图所示，现在存在两个物理寄存器r1，r2可以分配。

![basic_lra_1](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_lra_1.png)

对v1进行分配，此时active list为空，可以把r1物理寄存器分配给v1, 并将v1加入到active list当中。

对v2进行分配，因为v1的lifetime是从1到7的，所以v1依旧存在active当中，此时将物理寄存器r2分配给v2。

对v3进行分配，此时物理寄存器r1和r2都已经分配完，因此需要一个虚拟寄存器spill，因为v1的结束点最迟，因此选择v1进行spil，将v1从active list移除，将v3加入到active list当中。v3分配前后变化如下：

![basic_lra_2](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_lra_2.png)

对v4进行分配，此时v2生命周期结束，将v2从active当中移除，将v2的物理寄存器r2分配v4。

对v5进行分配，v3生命周期结束，将v3的物理寄存器r1分配给v5。
#### 2.3 缺陷：
由于basic算法的保守，要求所有的interval的生命周期都是连续的，不允许hole的存在，导致在分配的过程当中，v1不得不被spill到mem当中。分析指令后，发现v1在3被use后，在5被重新def了，所以中间存在一个hole，可以利用这个hole，先将v1的寄存器r1分配给v3,等v1需要的时候,v3也use完了，再分配给v1,进而避免spill，具体情况如下图所示：
![basic_lra_2](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_lra_3.png)

## 3 Optimized Interval Splitting In a Linear Scan Algorithm

#### 3.1 LRA算法的基本步骤
![basic_step](https://github.com/EchoWangHF/Blogs/blob/master/lra/basic_step.png)

#### 3.2 BB排序 （cnas这部分没有做特殊处理，直接调用BBList函数进行BB排序）
BB的排序算法在LRA当中非常重要，一个好的BB排序算法能够减少spill的次数，提高寄存器分配的效率，BB排序主要遵循下面的原则：

(1) 跳转指令链接的两个块尽可能地放在一起，以减少无关跳转的次数。

(2) 

(3) 

(4) 使用频率少的块尽可能放在后面，增加使用频率高的块的局部性。

##### 3.2.1 loop detection (是否存在更加高效的循环检测算法？需要进一步思考。)

仅针对loop head block为1的loop，对于loop head block 不为1的循环忽略，允许loop end block不为1的情况。

loop_index: 标记循环的unique number。

loop_depth: 表示该block被循环嵌套的层数，循环嵌套的层数越多，该block越重要。

###### 算法：

(1) collect the loop end blocks:

从CFG图的第一个BB和其后继(successor)开始，当BB第一次被访问的时候，标记为visited。当某一个BB的所有successor都被处理过了，则这个block被标记为active。当迭代算法再次到达一个被标记为active的block时，说明一个循环被找到了。该被标记为active的block被称为loop header，上一个被处理的block则是loop end。 创建一个loop_end_block 的list 存放知道的每一个loop end. 因为本算法只考虑loop header为1的情况，我们对每一个loop header分配一个loop index，用来标记该loop。当每一个block都被标记为visited时，整个迭代过程结束。

(2) mask all blocks of the loop:

遍历 loop end blocks的队列，以每一个loop end为起点，根据CFG图往前遍历,直到reached属于该loop的loop header。在遍历迭代的过程中，loop内的所有block都会被reached。构建二维的bitset, 存放block id 和被reached的 loop index。如果一个block 被多个loop reached，那么这个block会对应多个loop index。 每一个block的loop_depth的值，就时该block在bitset当中出现的累加和。每一个block的lopp_index，就是bitset该block id对的lowest loop。

###### 举例说明

![loop_detection](https://github.com/EchoWangHF/Blogs/blob/master/lra/loop_dete.PNG)

(1) CFG 图a展示了3 个loop 循环，loop end分别为B3、B7、B6，其中B3和B7的loop header相同，共用一个loop_index =1。loop end B6，其loop header为B4, 由于B4嵌套在B1，所以设置其index=0。

(2) 以loop_end_blocks list中的B7为起点，按照CFG图往前遍历，其中B4为直接前驱，B1, B2, B6为非直接前驱。由于B1本身为loop header，则B1不做处理，进而B0和B3也无法达到。由于B3在loop_end_blocks当中，所有B3最终会被标记，但是B0不会被标记。 最后的结果是： B0和B5没有被循环标记，B1、B2、B3、B7被loop_index=1标记了一次，B4和B6被loop_index=0和loop_index=1两个循环标记，共标记2次。

(3) loop_depth和loop_index 则可以算出来了，结果如c图所示。

##### 3.2.2 compute block order

(1) 构建两个队列，work_list和 block_order_list 。 work_list 主要处理CFG图当中的block， block_order_list存放最后的排序结果。block order主要按照权重(weight)的递增来进行排序的，而weight主要依据loop_depth，如果loop_depth的值相同，则基于其他的原则进行排序，例如异常处理的block则可以放在后面。对于一个有序的控制流，大部分的block的weight都是相同的，此时则按照stack的原理操作，后进先出。

(2) 首先将CFG图第一个block 放入wrok_list当中当作初始化，然后对work_list中的block进行遍历，即选择weight最高的block从work_list当中取出，放入到block_order_list当中，一直到work_list为空，遍历结束。

(3) 当一个block被处理的时候，其所有的后继block都将按照一定的规则，即：除了backward branch，所有的前驱block都被处理过了，相当于其ncoming forward branches为0。满足规则的，则放入到work_list当中。

具体的算法步骤如下：

![compute_block_order](https://github.com/EchoWangHF/Blogs/blob/master/lra/compute_block_order.JPG)

(4) 对于上图的示例，其block order如下图所示：

![compute_order_example](https://github.com/EchoWangHF/Blogs/blob/master/lra/block_order_1.PNG)

<1> 初始化，B0放入work_list当中，并对work_list当中的BB按照weight进行sort。

<2> pop work_list top的BB, 即：B0，将其放入block_ordre_list当中。当中B0被处理的时候，其所有的successor BB开始被遍历，看是否存在ncoming forward branches = 0的successor BB，如果存在，则将该BB丢进work_list当中。可以发现这里存在一个B1满足要求，从Figure 5.2 a)可以发现，虽然B1有三条入边，但是有两条来自B3和B7，属于backward branch，不受影响。一条来自B0，incoming forward branches=1, 但是B0处理过了，此时incoming forward branches=0,B1满足要求，放入到work_list当中，并sort。

<3> pop work_list top BB，即：B1, 将其放入到block_order_list当中。当B1被处理的时候，其successor BB B2和B3开始被遍历，这两个BB都满足要求，都可以被放入work_list当中，并且B2和B3的depth都是1, 因此weight也是1, sort也没有绝对的先后要求，这里我们可以将B2放在前面，先处理B2。

<4> pop work_list top BB，即：B2, 将其放入到block_order_list当中。当B2被处理的时候，其successor BB B4和B5开始被遍历，两者都符合要求，都可以被放入work_list当中。因为B5的weight为0, B4的weight为2, B3的weight为1, 则work_list sort后的结果为B5,B3,B4。

<5> pop work_list top BB，即：B4, 将其放入到block_order_list当中。B4的successor B7 和 B6 也都满足要求，可以被放入work_list。sort后，work_list当中的顺序为B5,B3, B7,B6。

<6> 按照上述的算法，最后block order的结果为：B0,B1,B2,B4,B6,B7,B3,B5。

##### 3.2.3 Numbering of IR
```c
int next_id = 0
for each block b in block_order_list do {
  for each op in b.operations do {
    op.id = next_id;
    next_id += 2;
  }
}
```

#### 3.3 Lifetime Intervals

lifetime interval是寄存器分配过程中的主要数据结构，其记录着寄存器的使用周期片段和其他信息。总体来说，在初始化的时候，每一个virtual register对应一个interval, 随着算法的进行，有的interval会被spilt，此时一个virtual register会对应多个interval。

interval的数据结构如下：

```c
class Interval {
  int reg_num; //表示interval 对应的virtual register号
  BasicType type; //表示分配到的物理寄存器的数据类型
  int assigned_reg; //物理寄存器号
  int assigned_regHi; //物理寄存器号：double数据类型可能需要两个物理寄存器
  //interval的生命周期的范围，range开始与def IR, 结束于最后一个use IR，左闭右开。
  //一个interval可以存在多个range，range之间的范围就是life of hole。
  RangeList ranges; 
  UsePositionList use_position; // 存储寄存器use IR的id_number。
  //当interval在某个阶段被spill或者reload的时候，其locationf发生了变化，从memory变为register，或者从register变为memory，
  //为了保持location统一，会将interval spilt。新的interval称为spilt_children, 原先的interval称为spilt_parent。
  //spilt_position 之后range移到spilt_children, 之前的部分保留在spilt_parent。
  Interval spilt_parent;
  Interval spilt_children;
  Interval register_hint;
};

class Range {
  int from;
  int to;
}

class UsePosition {
  int position;
  int use_kind;
}
```







