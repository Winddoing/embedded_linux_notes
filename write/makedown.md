# Makedown的一些常用语法


## 代码高亮

* ``` C
*  printf("hello word!\n");
* ```

效果：

``` C
 printf("hello word!\n");
```

## 流程图

* ```flow
* st=>start: Start:>https://www.zybuluo.com
* io=>inputoutput: verification
* op=>operation: Your Operation
* cond=>condition: Yes or No?
* sub=>subroutine: Your Subroutine
* e=>end

* st->io->op->cond
* cond(yes)->e
* cond(no)->sub->io
* ```

效果：

```flow
st=>start: Start:>https://www.zybuluo.com
io=>inputoutput: verification
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
```

## 强调字或词

使用\`...\`

用于表明一句话中的\`关键字\`和\`词\`

效果：

用于表明一句话中的`关键字`和`词`

## 加粗

文字两端使用2个“*”或者“_”夹起来

将需要设置为斜体的文字两端使用1个“*”或者“_”夹起来

## 图片

``` 
![Alt text](/path/to/img.jpg)

![Alt text](/path/to/img.jpg "Optional title")
```
