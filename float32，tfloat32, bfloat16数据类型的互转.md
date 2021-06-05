在深度学习领域，NVIDIA提出了BFloat16 和 TFloat32两种新的特殊的数据了类型，这两种数据的具体定义和优势可以参考英伟达的这个介绍：https://blogs.nvidia.com/blog/2020/05/14/tensorfloat-32-precision-format/ 。

对于Float32，BFloat16 和 TFloat32 三种数据类型的除了符号位1位，指数位都是相同的，都是8位。区别在于尾数位，float32的尾数为23位，tfloat32的尾数位为10位(tensor float 又可以称为float19)，bfloat16的的尾数位为7位，因此转换比较简单。而float16数据类型则是1位符号位，5位指数位，10位尾数位，由于指数位的不同，转换起来就比较麻烦了。

下面是一段从tensorflow源代码里面截取的一段float32转bfloat16的代码，从代码里面可以看到float32转bfloat16就是尾数位直接截断，同理我们可以推出float32转tfloat32, 以及tfloat32转bfloat16都是尾数位直接阶段即可。

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
上述代码
