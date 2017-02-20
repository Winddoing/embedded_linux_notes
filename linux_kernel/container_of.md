# 宏container_of的实现原理


## 定义:

``` C
/**  
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:    the pointer to the member.
 * @type:   the type of the container struct this is embedded in.
 * @member: the name of the member within the struct.
 *   
 */
#define container_of(ptr, type, member) ({          \                                                              
    const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```

## 作用:

>用于从包含在某个结构中的指针获得结构体本身的指针，通俗地讲就是通过结构体变量中某个成员的首地址进而获得整个结构体变量的首地址.

## 原理:

1. 首先定义一个临时的数据类型（通过typeof( ((type *)0)->member )获得）与ptr相同的指针变量__mptr，然后用它来保存ptr的值。

2. 用(char *)__mptr减去member在结构体中的偏移量，得到的值就是整个结构体变量的首地址（整个宏的返回值就是这个首地址）。

## 示例:

``` C
/* linux-2.6.38.8/include/linux/compiler-gcc4.h */
#define __compiler_offsetof(a,b) __builtin_offsetof(a,b)

/* linux-2.6.38.8/include/linux/stddef.h */
#undef offsetof
#ifdef __compiler_offsetof
#define offsetof(TYPE,MEMBER) __compiler_offsetof(TYPE,MEMBER)
#else
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
#endif

/* linux-2.6.38.8/include/linux/kernel.h *
 * container_of - cast a member of a structure out to the containing structure
 * @ptr: the pointer to the member.
 * @type:	the type of the container struct this is embedded in.
 * @member:    the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({	    \
	const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
	(type *)( (char *)__mptr - offsetof(type,member) );})

#include <stdio.h>

struct test_struct {
    int num;
    char ch;
    float fl;
};

int main(void)
{
    struct test_struct init_test_struct = { 99, 'C', 59.12 };

    char *char_ptr = &init_test_struct.ch;

    struct test_struct *test_struct = container_of(char_ptr, struct test_struct, ch);
	/* struct test_struct *test_struct = container_of(char_ptr, struct test_struct, ch); //error */
    
    printf(" test_struct->num = %d\n test_struct->ch = %c\n test_struct->fl = %f\n", 
	    test_struct->num, test_struct->ch, test_struct->fl);
    
    return 0;
}
```

## 参考:

1. [ Linux内核中的常用宏container_of](http://blog.csdn.net/npy_lp/article/details/7010752)
