在深度学习领域，`NVIDIA`提出了`BFloat16`和`TFloat32`两种新的特殊的数据了类型，这两种数据的具体定义和优势可以参考英伟达的这个介绍：https://blogs.nvidia.com/blog/2020/05/14/tensorfloat-32-precision-format/ 。

对于`Float32`，`BFloat16`和`TFloat32`三种数据类型的除了符号位`1`位，指数位都是相同的，都是`8`位。区别在于尾数位，`float32`的尾数为`23`位，`tfloat32`的尾数位为`10`位(`tensor float`又可以称为`float19`)，`bfloat16`的的尾数位为7位，因此转换比较简单。而`float16`数据类型则是`1`位符号位，`5`位指数位，`10`位尾数位，由于指数位的不同，转换起来就比较麻烦了。

下面是一段从`tensorflow`源代码里面截取的一段`float32`转`bfloat16`的代码，从代码里面可以看到`float32`转`bfloat16`就是尾数位直接截断，同理我们可以推出`float32`转`tfloat32`, 以及`tfloat32`转`bfloat16`都是尾数位直接阶段即可。

```c
 void FloatToBFloat16(const float* src, bfloat16* dst, int64 size) {
   for (; size != 0; src++, dst++, size--) {
 #if __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__
     memcpy(dst, src, sizeof(bfloat16));
 #else
     memcpy(
         dst,
         reinterpret_cast<const char*>(src) + sizeof(float) - sizeof(bfloat16),
         sizeof(bfloat16));
 #endif
   }
 }
```
上述代码当中涉及到一个概念，就是大端存储和小端存储的问题，小端就是数据的低字节存储到低地址当中，高字节存储到高地址当中，大端存储正好相反。

假如有一个数据是`0x12345678`, 存储到地址位`0x200~0x2003`内存当中，小端存储如下：
```math
0x200   0x201   0x202   0x203
0x78    0x56    0x34    0x12
```
如果需要截取数据将`0x12345678` 截取为`0x123456`，那么正好是从`0x201`开始取数，做一下起始位置的偏移，所以代码当中才会做那样的`sizeof(float) - sizeof(bfloat16)`的偏移操作。

如果将`bfloat16`转成`float32`也是类似`memcpy`数据，但是目的地址需要做类似的偏移，从`0x201`开始存放数据，并且在`0x200`的位置置`0`。

`tfloat32`数据的转换与上述类似，不做赘述。
