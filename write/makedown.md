# Makedown的一些常用语法

## 代码高亮
```
``` C
 printf("hello word!\n");
```
```
效果：
``` C
 printf("hello word!\n");
```

## 流程图

```
```
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
```
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

用于表明一句话中的关键字和词，使用\`...\`
效果：
用于表明一句话中的`关键字`和`词`
