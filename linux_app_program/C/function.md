# 函数


``` C
#include<stdio.h>                              
                                               
void test()                                    
{                                              
    printf("hello world!\n");                  
}                                              
                                               
int main(int argc, char *argv[])               
{                                              
    printf("0x%08x\n",test);                   
    printf("0x%08x\n",&test);                  
}                                              
```
执行结果:
```
=====>$./a.out 
0x0040057d
0x0040057d
```
> **二者地址相同但是类型不同**

## 函数地址

> 表示`test`函数的首地址，它的类型是`void ()`


## 函数名的地址

> 表示一个指向函数`test`这个对象的地址, 类型是`void (*)()`





