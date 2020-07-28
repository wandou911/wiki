# iOS使用MLeaksFinder内存检测

工具 MLeaksFinder

[MLeaksFinder](https://github.com/Tencent/MLeaksFinder)

## 成员变量引起的循环引用

在Block的使用当中，当self强持有一个Blcok的时候，同时在Block内部也去强持有self的时候，那么就会造成在self无法释放，也是就是说造成了内存泄漏，这便是循环引用的一个问题。至于解决的办法则是用弱引用去打破这么一个闭环。代码如下：
`__weak typeof(self) weakSelf = self;`

而当使用成员变量的时候 因为没有通过点语法去获取的。那么我们怎么给当前成员使用self去获取呢！这个时候我们就可以通过self->的方式就实现self的调用.然后结合上面的__weak就可实现成员变量在循环引用中的问题解决。




