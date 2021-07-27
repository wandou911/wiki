

1.block是什么？block是对象吗？

闭包是一个函数（或指向函数的指针），再加上该函数执行的外部的上下文变量（有时候也称作自由变量）。

block 实际上就是 Objective-C 语言对于闭包的实现。

下图是block的数据结构定义，显而易见，在Block_layout里，我们看到了isa指针
为什么说block是对象呢，原因就在于isa指针。那么这个isa指针是何物呢？

所有对象的都有isa 指针，用于实现对象相关的功能。

2.block分为哪几种？__blcok关键字的作用？

分为三种，即NSConcreteGlobalBlock、NSConcreteStackBlock、NSConcreteMallocBlock。

首先是NSConcreteGlobalBlock：

如果一个block钟没有引用外部变量


```
void (^block1)(void) = ^{
        int i = 1024;
        printf("%d\n", i);
    };
    block1();
    NSLog(@"%@", block1);
```

什么是NSConcreteStackBlock呢：

可以这么理解，NSConcreteStackBlock就是引用了外部变量的block，上代码：

```
int i = 1024;
    void (^block1)(void) = ^{

        printf("%d\n", i);
    };
    block1();
    NSLog(@"%@", block1);
```

3.block在ARC和MRC下的区别？

这是因为本身我们常常将block赋值给变量，而ARC下默认的赋值操作是strong的，到了block身上自然就成了copy，所以常常打印出来的block就是NSConcreteMallocBlock了。

4.block的生命周期？

我是如何来处理block的生命周期相关问题的呢，首先前文提到，block是一个对象，既然是一个对象，它必然有着和对象一样的生命周期即如果没有被引用就会被释放。

所以block的生命周期归结起来很简单，只要看持有block的对象是不是也被block持有，如果没有持有，就不用担心循环引用问题了。

5.block对于以参数形式传进来的对象，会不会强引用？？

其实block与函数和方法一样，对于传进来的参数，并不会持有

#### 参考链接

1 [让我们来深入浅出block吧/](http://kuailejim.com/2016/04/22/让我们来深入浅出block吧/)

2 [谈Objective-C block的实现](http://blog.devtang.com/2013/07/28/a-look-inside-blocks/)
