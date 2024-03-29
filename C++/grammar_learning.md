# C++ 语法的心得领悟
## 1 `const T*` 和 `T* const` 的区别
一句话概括就是：`const T*` 是地址中的值不能变，但是指针地址可以变化。 `T* const` 是地址当中的值可以变，但是地址不能变。

case 1：
```c++
const int32_t* a = new int(1);                                                
const int32_t* b = new int(2);                                                
std::cout<<"before: a : " << *a << " b : " << *b << std::endl;                
std::cout<<"before: a addr : " << a << " b addr: " << b << std::endl;         
a = b;  // 合法，地址修改，将b的地址赋给a，a的地址发生了变化。  
// *a = 3 //  不合法， 对a地址当中的值进行了修改
std::cout<<"after:  a : " << *a << " b : " << *b << std::endl;                   
std::cout<<"after: a addr : " << a << " b addr: " << b << std::endl;          
```
```
before: a : 1 b : 2
before: a addr : 0x55cda9063eb0 b addr: 0x55cda9063ed0
after:  a : 2 b : 2
after: a addr : 0x55cda9063ed0 b addr: 0x55cda9063ed0
```
case 2： 
```c++
int32_t* const a = new int(1);                                                
int32_t* const b = new int(2);                 
std::cout<<"before: a : " << *a << " b : " << *b << std::endl;
std::cout<<"before: a addr : " << a << " b addr: " << b << std::endl; 
*a = 3; // 合法，对地址当中的值进行修改，地址不做修改。
// a = b; // 不合法，对地址进行了修改。             
std::cout<<"after:  a : " << *a << " b : " << *b << std::endl;  
std::cout<<"after: a addr : " << a << " b addr: " << b << std::endl; 
```
```
before: a : 1 b : 2
before: a addr : 0x5644b48c1eb0 b addr: 0x5644b48c1ed0
after:  a : 3 b : 2
after: a addr : 0x5644b48c1eb0 b addr: 0x5644b48c1ed0
```
记忆技巧： `const int* a`, 一般`int* a`是取地址当中的值，所以`const`是对地址当中的值进行限制。`int* const a`, 一般`a` 是取地址的，所以`const`是对地址进行限制。
