在深度学习领域，NVIDIA提出了BFloat16 和 TFloat32两种新的特殊的数据了类型，这两种数据的具体定义和优势可以参考英伟达的这个介绍：https://blogs.nvidia.com/blog/2020/05/14/tensorfloat-32-precision-format/。

相比传统的Float32数据类型，由于BFloat16 和 TFloat32 两种数据类型的指数位都是8位，区别在于小数位，因此转换比较简单。

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
