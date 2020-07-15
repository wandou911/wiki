iOS多线程
参考链接[iOS多线程](http://www.cocoachina.com/ios/20170707/19769.html)

## 目的

本文主要是分享iOS多线程的相关内容，为了更系统的讲解，将分为以下7个方面来展开描述。

1 多线程的基本概念
2 线程的状态与生命周期
3 多线程的四种解决方案：pthread，NSThread，GCD，NSOperation
4 线程安全问题
5 NSThread的使用
6 GCD的理解与使用
7 NSOperation的理解与使用

Demo在这里：[WHMultiThreadDemo](https://github.com/remember17/WHMultiThreadDemo)

Demo 效果图

![demo](http://cc.cocimg.com/api/uploads/20170707/1499394574376547.gif)

## 一、多线程的基本概念

* 进程：可以理解成一个运行中的应用程序，是系统进行资源分配和调度的基本单位，是操作系统结构的基础，主要管理资源。
* 线程：是进程的基本执行单元，一个进程对应多个线程。
* 主线程：处理UI，所有更新UI的操作都必须在主线程上执行。不要把耗时操作放在主线程，会卡界面。
* 多线程：在同一时刻，一个CPU只能处理1条线程，但CPU可以在多条线程之间快速的切换，只要切换的足够快，就造成了多线程一同执行的假象。
* 线程就像火车的一节车厢，进程则是火车。车厢（线程）离开火车（进程）是无法跑动的，而火车（进程）至少有一节车厢（主线程）。多线程可以看做多个车厢，它的出现是为了提高效率。
* 多线程是通过提高资源使用率来提高系统总体的效率。
* 我们运用多线程的目的是：将耗时的操作放在后台执行！
## 二、线程的状态与生命周期

下图是线程状态示意图，从图中可以看出线程的生命周期是：新建 - 就绪 - 运行 - 阻塞 - 死亡

![线程状态](http://cc.cocimg.com/api/uploads/20170707/1499394752139363.png)

下面分别阐述线程生命周期中的每一步

* 新建：实例化线程对象
* 就绪：向线程对象发送start消息，线程对象被加入可调度线程池等待CPU调度。
* 运行：CPU 负责调度可调度线程池中线程的执行。线程执行完成之前，状态可能会在就绪和运行之间来回切换。就绪和运行之间的状态变化由CPU负责，程序员不能干预。
* 阻塞：当满足某个预定条件时，可以使用休眠或锁，阻塞线程执行。sleepForTimeInterval（休眠指定时长），sleepUntilDate（休眠到指定日期），@synchronized(self)：（互斥锁）。
* 死亡：正常死亡，线程执行完毕。非正常死亡，当满足某个条件后，在线程内部中止执行/在主线程中止线程对象
还有线程的exit和cancel
* [NSThread exit]：一旦强行终止线程，后续的所有代码都不会被执行。
* [thread cancel]取消：并不会直接取消线程，只是给线程对象添加 isCancelled 标记。

## 三、多线程的四种解决方案

多线程的四种解决方案分别是：pthread，NSThread，GCD， NSOperation。

下图是对这四种方案进行了解读和对比。

![对比](http://cc.cocimg.com/api/uploads/20170707/1499394732413995.png)

## 四、线程安全问题

当多个线程访问同一块资源时，很容易引发数据错乱和数据安全问题。就好比几个人在同一时修改同一个表格，造成数据的错乱。

解决多线程安全问题的方法

* 方法一：互斥锁（同步锁）

```
@synchronized(锁对象) {
    // 需要锁定的代码
}
```

判断的时候锁对象要存在，如果代码中只有一个地方需要加锁，大多都使用self作为锁对象，这样可以避免单独再创建一个锁对象。

加了互斥做的代码，当新线程访问时，如果发现其他线程正在执行锁定的代码，新线程就会进入休眠。


* 方法二：自旋锁
加了自旋锁，当新线程访问代码时，如果发现有其他线程正在锁定代码，新线程会用死循环的方式，一直等待锁定的代码执行完成。相当于不停尝试执行代码，比较消耗性能。

属性修饰atomic本身就有一把自旋锁。

下面说一下属性修饰nonatomic 和 atomic

```
nonatomic 非原子属性,同一时间可以有很多线程读和写
atomic 原子属性(线程安全)，保证同一时间只有一个线程能够写入(但是同一个时间多个线程都可以取值)，atomic 本身就有一把锁(自旋锁)
atomic：线程安全，需要消耗大量的资源
nonatomic：非线程安全，不过效率更高，一般使用nonatomic
```

## 五、NSThread的使用

### No.1：NSThread创建线程

NSThread有三种创建方式：

* init方式
* detachNewThreadSelector创建好之后自动启动
* performSelectorInBackground创建好之后也是直接启动

```
/** 方法一，需要start */
NSThread *thread1 = [[NSThread alloc] initWithTarget:self selector:@selector(doSomething1:) object:@"NSThread1"];
// 线程加入线程池等待CPU调度，时间很快，几乎是立刻执行
[thread1 start];
 
/** 方法二，创建好之后自动启动 */
[NSThread detachNewThreadSelector:@selector(doSomething2:) toTarget:self withObject:@"NSThread2"];
 
/** 方法三，隐式创建，直接启动 */
[self performSelectorInBackground:@selector(doSomething3:) withObject:@"NSThread3"];
 
- (void)doSomething1:(NSObject *)object {
    // 传递过来的参数
    NSLog(@"%@",object);
    NSLog(@"doSomething1：%@",[NSThread currentThread]);
}
 
- (void)doSomething2:(NSObject *)object {
    NSLog(@"%@",object);
    NSLog(@"doSomething2：%@",[NSThread currentThread]);
}
 
- (void)doSomething3:(NSObject *)object {
    NSLog(@"%@",object);
    NSLog(@"doSomething3：%@",[NSThread currentThread]);
}
```

### No.2：NSThread的类方法

* 返回当前线程

```
// 当前线程
[NSThread currentThread];
NSLog(@"%@",[NSThread currentThread]);
 
// 如果number=1，则表示在主线程，否则是子线程
打印结果：{number = 1, name = main}
```

* 阻塞休眠

```
//休眠多久
[NSThread sleepForTimeInterval:2];
//休眠到指定时间
[NSThread sleepUntilDate:[NSDate date]];
```

* 类方法补充

```
//退出线程
[NSThread exit];
//判断当前线程是否为主线程
[NSThread isMainThread];
//判断当前线程是否是多线程
[NSThread isMultiThreaded];
//主线程的对象
NSThread *mainThread = [NSThread mainThread];
```

### No.3：NSThread的一些属性

```
//线程是否在执行
thread.isExecuting;
//线程是否被取消
thread.isCancelled;
//线程是否完成
thread.isFinished;
//是否是主线程
thread.isMainThread;
//线程的优先级，取值范围0.0到1.0，默认优先级0.5，1.0表示最高优先级，优先级高，CPU调度的频率高
 thread.threadPriority;
```


## 六、GCD的理解与使用

### No.1：GCD的特点

* GCD会自动利用更多的CPU内核
* GCD自动管理线程的生命周期（创建线程，调度任务，销毁线程等）
* 程序员只需要告诉 GCD 想要如何执行什么任务，不需要编写任何线程管理代码

### No.2：GCD的基本概念

* 任务（block）：任务就是将要在线程中执行的代码，将这段代码用block封装好，然后将这个任务添加到指定的执行方式（同步执行和异步执行），等待CPU从队列中取出任务放到对应的线程中执行。
* 同步（sync）：一个接着一个，前一个没有执行完，后面不能执行，不开线程。
* 异步（async）：开启多个新线程，任务同一时间可以一起执行。异步是多线程的代名词
* 队列：装载线程任务的队形结构。(系统以先进先出的方式调度队列中的任务执行)。在GCD中有两种队列：串行队列和并发队列。
* 并发队列：线程可以同时一起进行执行。实际上是CPU在多条线程之间快速的切换。（并发功能只有在异步（dispatch_async）函数下才有效）
* 串行队列：线程只能依次有序的执行。
* GCD总结：将任务(要在线程中执行的操作block)添加到队列(自己创建或使用全局并发队列)，并且指定执行任务的方式(异步dispatch_async，同步dispatch_sync)

### No.3：队列的创建方法

* 使用dispatch_queue_create来创建队列对象，传入两个参数，第一个参数表示队列的唯一标识符，可为空。第二个参数用来表示串行队列（DISPATCH_QUEUE_SERIAL）或并发队列（DISPATCH_QUEUE_CONCURRENT）。

```
/ 串行队列
dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_SERIAL);
// 并发队列
dispatch_queue_t queue1 = dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);
```

* GCD的队列还有另外两种

主队列：主队列负责在主线程上调度任务，如果在主线程上已经有任务正在执行，主队列会等到主线程空闲后再调度任务。通常是返回主线程更新UI的时候使用。dispatch_get_main_queue()

```
dispatch_async(dispatch_get_global_queue(0, 0), ^{
      // 耗时操作放在这里
      
      dispatch_async(dispatch_get_main_queue(), ^{
          // 回到主线程进行UI操作

      });
  });
```
全局并发队列：全局并发队列是就是一个并发队列，是为了让我们更方便的使用多线程。
dispatch_get_global_queue(0, 0)

```
//全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
//全局并发队列的优先级
#define DISPATCH_QUEUE_PRIORITY_HIGH 2 // 高优先级
#define DISPATCH_QUEUE_PRIORITY_DEFAULT 0 // 默认（中）优先级
#define DISPATCH_QUEUE_PRIORITY_LOW (-2) // 低优先级
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN // 后台优先级
//iOS8开始使用服务质量，现在获取全局并发队列时，可以直接传0
dispatch_get_global_queue(0, 0);
```

### No.4：同步/异步/任务、创建方式

同步（sync）使用dispatch_sync来表示。

异步（async）使用dispatch_async。

任务就是将要在线程中执行的代码，将这段代码用block封装好。

代码如下：

```
// 同步执行任务
    dispatch_sync(dispatch_get_global_queue(0, 0), ^{
        // 任务放在这个block里
        NSLog(@"我是同步执行的任务");
 
    });
    // 异步执行任务
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 任务放在这个block里
        NSLog(@"我是异步执行的任务");
 
    });
```

No.5：GCD的使用

由于有多种队列（串行/并发/主队列）和两种执行方式（同步/异步），所以他们之间可以有多种组合方式。

1 串行同步
2 串行异步
3 并发同步
4 并发异步
5 主队列同步
6 主队列异步

* 串行同步

执行完一个任务，再执行下一个任务。不开启新线程。

```
/** 串行同步 */
- (void)syncSerial {
 
    NSLog(@"\n\n**************串行同步***************\n\n");
 
    // 串行队列
    dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_SERIAL);
 
    // 同步执行
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"串行同步1   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"串行同步2   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"串行同步3   %@",[NSThread currentThread]);
        }
    });
}
```

输入结果为顺序执行，都在主线程：

```
串行同步1   {number = 1, name = main}
串行同步1   {number = 1, name = main}
串行同步1   {number = 1, name = main}
串行同步2   {number = 1, name = main}
串行同步2   {number = 1, name = main}
串行同步2   {number = 1, name = main}
串行同步3   {number = 1, name = main}
串行同步3   {number = 1, name = main}
串行同步3   {number = 1, name = main}
```

* 串行异步

开启新线程，但因为任务是串行的，所以还是按顺序执行任务。

```
/** 串行异步 */
- (void)asyncSerial {
 
    NSLog(@"\n\n**************串行异步***************\n\n");
 
    // 串行队列
    dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_SERIAL);
 
    // 同步执行
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"串行异步1   %@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"串行异步2   %@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"串行异步3   %@",[NSThread currentThread]);
        }
    });
}
```

输入结果为顺序执行，有不同线程：

```
串行异步1   {number = 3, name = (null)}
串行异步1   {number = 3, name = (null)}
串行异步1   {number = 3, name = (null)}
串行异步2   {number = 3, name = (null)}
串行异步2   {number = 3, name = (null)}
串行异步2   {number = 3, name = (null)}
串行异步3   {number = 3, name = (null)}
串行异步3   {number = 3, name = (null)}
串行异步3   {number = 3, name = (null)}
```

* 并发同步

因为是同步的，所以执行完一个任务，再执行下一个任务。不会开启新线程。

```
/** 并发同步 */
- (void)syncConcurrent {
3
    NSLog(@"\n\n**************并发同步***************\n\n");
3
    // 并发队列
    dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);
3
    // 同步执行
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"并发同步1   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"并发同步2   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"并发同步3   %@",[NSThread currentThread]);
        }
    });
}
```

输入结果为顺序执行，都在主线程：

```
并发同步1   {number = 1, name = main}
并发同步1   {number = 1, name = main}
并发同步1   {number = 1, name = main}
并发同步2   {number = 1, name = main}
并发同步2   {number = 1, name = main}
并发同步2   {number = 1, name = main}
并发同步3   {number = 1, name = main}
并发同步3   {number = 1, name = main}
并发同步3   {number = 1, name = main}
```

* 并发异步

任务交替执行，开启多线程。

```
/** 并发异步 */
- (void)asyncConcurrent {
 
    NSLog(@"\n\n**************并发异步***************\n\n");
 
    // 并发队列
    dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);
 
    // 同步执行
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"并发异步1   %@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"并发异步2   %@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"并发异步3   %@",[NSThread currentThread]);
        }
    });
}
```

输入结果为无序执行，有多条线程：

```
并发异步1   {number = 3, name = (null)}
并发异步2   {number = 4, name = (null)}
并发异步3   {number = 5, name = (null)}
并发异步1   {number = 3, name = (null)}
并发异步2   {number = 4, name = (null)}
并发异步3   {number = 5, name = (null)}
并发异步1   {number = 3, name = (null)}
并发异步2   {number = 4, name = (null)}
并发异步3   {number = 5, name = (null)}
```

* 主队列同步

如果在主线程中运用这种方式，则会发生死锁，程序崩溃。

```
/** 主队列同步 */
- (void)syncMain {
 
    NSLog(@"\n\n**************主队列同步，放到主线程会死锁***************\n\n");
 
    // 主队列
    dispatch_queue_t queue = dispatch_get_main_queue();
 
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"主队列同步1   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"主队列同步2   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"主队列同步3   %@",[NSThread currentThread]);
        }
    });
}
```

主队列同步造成死锁的原因：

1 如果在主线程中运用主队列同步，也就是把任务放到了主线程的队列中。
2 而同步对于任务是立刻执行的，那么当把第一个任务放进主队列时，它就会立马执行。
3 可是主线程现在正在处理syncMain方法，任务需要等syncMain执行完才能执行。
4 syncMain执行到第一个任务的时候，又要等第一个任务执行完才能往下执行第二个和第三个任务。
5 这样syncMain方法和第一个任务就开始了互相等待，形成了死锁。

* 主队列异步

```
/** 主队列异步 */
- (void)asyncMain {
 
    NSLog(@"\n\n**************主队列异步***************\n\n");
 
    // 主队列
    dispatch_queue_t queue = dispatch_get_main_queue();
 
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"主队列异步1   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"主队列异步2   %@",[NSThread currentThread]);
        }
    });
    dispatch_sync(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"主队列异步3   %@",[NSThread currentThread]);
        }
    });
}
```

输入结果为在主线程中按顺序执行：

```
主队列异步1   {number = 1, name = main}
主队列异步1   {number = 1, name = main}
主队列异步1   {number = 1, name = main}
主队列异步2   {number = 1, name = main}
主队列异步2   {number = 1, name = main}
主队列异步2   {number = 1, name = main}
主队列异步3   {number = 1, name = main}
主队列异步3   {number = 1, name = main}
主队列异步3   {number = 1, name = main}
```

* GCD线程之间的通讯

开发中需要在主线程上进行UI的相关操作，通常会把一些耗时的操作放在其他线程，比如说图片文件下载等耗时操作。

当完成了耗时操作之后，需要回到主线程进行UI的处理，这里就用到了线程之间的通讯。

```
- (IBAction)communicationBetweenThread:(id)sender {
 
    // 异步
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 耗时操作放在这里，例如下载图片。（运用线程休眠两秒来模拟耗时操作）
        [NSThread sleepForTimeInterval:2];
        NSString *picURLStr = @"http://www.bangmangxuan.net/uploads/allimg/160320/74-160320130500.jpg";
        NSURL *picURL = [NSURL URLWithString:picURLStr];
        NSData *picData = [NSData dataWithContentsOfURL:picURL];
        UIImage *image = [UIImage imageWithData:picData];
 
        // 回到主线程处理UI
        dispatch_async(dispatch_get_main_queue(), ^{
            // 在主线程上添加图片
            self.imageView.image = image;
        });
    });
}
```

上面的代码是在新开的线程中进行图片的下载，下载完成之后回到主线程显示图片。

* GCD栅栏

当任务需要异步进行，但是这些任务需要分成两组来执行，第一组完成之后才能进行第二组的操作。这时候就用了到GCD的栅栏方法dispatch_barrier_async。

```
- (IBAction)barrierGCD:(id)sender {
 
    // 并发队列
    dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);
 
    // 异步执行
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"栅栏：并发异步1   %@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"栅栏：并发异步2   %@",[NSThread currentThread]);
        }
    });
 
    dispatch_barrier_async(queue, ^{
        NSLog(@"------------barrier------------%@", [NSThread currentThread]);
        NSLog(@"******* 并发异步执行，但是34一定在12后面 *********");
    });
 
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"栅栏：并发异步3   %@",[NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"栅栏：并发异步4   %@",[NSThread currentThread]);
        }
    });
}
```

上面代码的打印结果如下，开启了多条线程，所有任务都是并发异步进行。但是第一组完成之后，才会进行第二组的操作。

```
栅栏：并发异步1   {number = 3, name = (null)}
栅栏：并发异步2   {number = 6, name = (null)}
栅栏：并发异步1   {number = 3, name = (null)}
栅栏：并发异步2   {number = 6, name = (null)}
栅栏：并发异步1   {number = 3, name = (null)}
栅栏：并发异步2   {number = 6, name = (null)}
 ------------barrier------------{number = 6, name = (null)}
******* 并发异步执行，但是34一定在12后面 *********
栅栏：并发异步4   {number = 3, name = (null)}
栅栏：并发异步3   {number = 6, name = (null)}
栅栏：并发异步4   {number = 3, name = (null)}
栅栏：并发异步3   {number = 6, name = (null)}
栅栏：并发异步4   {number = 3, name = (null)}
栅栏：并发异步3   {number = 6, name = (null)}
```

* GCD延时执行

当需要等待一会再执行一段代码时，就可以用到这个方法了：dispatch_after。

```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    // 5秒后异步执行
    NSLog(@"我已经等待了5秒！");
});
GCD实现代码只执行一次
使用dispatch_once能保证某段代码在程序运行过程中只被执行1次。可以用来设计单例。
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    NSLog(@"程序运行过程中我只执行了一次！");
});
```

* GCD快速迭代

GCD有一个快速迭代的方法dispatch_apply，dispatch_apply可以同时遍历多个数字。

```
- (IBAction)applyGCD:(id)sender {
 
    NSLog(@"\n\n************** GCD快速迭代 ***************\n\n");
 
    // 并发队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
 
    // dispatch_apply几乎同时遍历多个数字
    dispatch_apply(7, queue, ^(size_t index) {
        NSLog(@"dispatch_apply：%zd======%@",index, [NSThread currentThread]);
    });
}
```

打印结果如下：

```
dispatch_apply：0======{number = 1, name = main}
dispatch_apply：1======{number = 1, name = main}
dispatch_apply：2======{number = 1, name = main}
dispatch_apply：3======{number = 1, name = main}
dispatch_apply：4======{number = 1, name = main}
dispatch_apply：5======{number = 1, name = main}
dispatch_apply：6======{number = 1, name = main}
```

* GCD队列组

异步执行几个耗时操作，当这几个操作都完成之后再回到主线程进行操作，就可以用到队列组了。

队列组有下面几个特点：

1 所有的任务会并发的执行(不按序)。
2 所有的异步函数都添加到队列中，然后再纳入队列组的监听范围。
3 使用dispatch_group_notify函数，来监听上面的任务是否完成，如果完成, 就会调用这个方法。

队列组示例代码：

```
- (void)testGroup {
    dispatch_group_t group =  dispatch_group_create();
 
    dispatch_group_async(group, dispatch_get_global_queue(0, 0), ^{
        NSLog(@"队列组：有一个耗时操作完成！");
    });
 
    dispatch_group_async(group, dispatch_get_global_queue(0, 0), ^{
        NSLog(@"队列组：有一个耗时操作完成！");
    });
 
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"队列组：前面的耗时操作都完成了，回到主线程进行相关操作");
    });
}
```

打印结果如下：

```
队列组：有一个耗时操作完成！
队列组：有一个耗时操作完成！
队列组：前面的耗时操作都完成了，回到主线程进行相关操作
```

至此，GCD的相关内容叙述完毕。下面让我们继续学习NSOperation。

## 七、NSOperation的理解与使用

### No.1：NSOperation简介

NSOperation是基于GCD之上的更高一层封装，NSOperation需要配合NSOperationQueue来实现多线程。

NSOperation实现多线程的步骤如下：

```
1. 创建任务：先将需要执行的操作封装到NSOperation对象中。
2. 创建队列：创建NSOperationQueue。
3. 将任务加入到队列中：将NSOperation对象添加到NSOperationQueue中。
```

需要注意的是，NSOperation是个抽象类，实际运用时中需要使用它的子类，有三种方式：

1 使用子类NSInvocationOperation
2 使用子类NSBlockOperation
3 定义继承自NSOperation的子类，通过实现内部相应的方法来封装任务。

### No.2：NSOperation的三种创建方式

* NSInvocationOperation的使用

创建NSInvocationOperation对象并关联方法，之后start。

```
- (void)testNSInvocationOperation {
    // 创建NSInvocationOperation
    NSInvocationOperation *invocationOperation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationOperation) object:nil];
    // 开始执行操作
    [invocationOperation start];
}
 
- (void)invocationOperation {
    NSLog(@"NSInvocationOperation包含的任务，没有加入队列========%@", [NSThread currentThread]);
}
```

打印结果如下，得到结论：程序在主线程执行，没有开启新线程。

这是因为NSOperation多线程的使用需要配合队列NSOperationQueue，后面会讲到NSOperationQueue的使用。

```
NSInvocationOperation包含的任务，没有加入队列========{number = 1, name = main}

```

* NSBlockOperation的使用

把任务放到NSBlockOperation的block中，然后start。

```
- (void)testNSBlockOperation {
    // 把任务放到block中
    NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"NSBlockOperation包含的任务，没有加入队列========%@", [NSThread currentThread]);
    }];
 
    [blockOperation start];
}
```

执行结果如下，可以看出：主线程执行，没有开启新线程。

同样的，NSBlockOperation可以配合队列NSOperationQueue来实现多线程。

```
NSBlockOperation包含的任务，没有加入队列========{number = 1, name = main}

```

但是NSBlockOperation有一个方法addExecutionBlock:，通过这个方法可以让NSBlockOperation实现多线程。

```
- (void)testNSBlockOperationExecution {
    NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"NSBlockOperation运用addExecutionBlock主任务========%@", [NSThread currentThread]);
    }];
 
    [blockOperation addExecutionBlock:^{
        NSLog(@"NSBlockOperation运用addExecutionBlock方法添加任务1========%@", [NSThread currentThread]);
    }];
    [blockOperation addExecutionBlock:^{
        NSLog(@"NSBlockOperation运用addExecutionBlock方法添加任务2========%@", [NSThread currentThread]);
    }];
    [blockOperation addExecutionBlock:^{
        NSLog(@"NSBlockOperation运用addExecutionBlock方法添加任务3========%@", [NSThread currentThread]);
    }];
    [blockOperation start];
}
```

执行结果如下，可以看出，NSBlockOperation创建时block中的任务是在主线程执行，而运用addExecutionBlock加入的任务是在子线程执行的。

```
NSBlockOperation运用addExecutionBlock========{number = 1, name = main}
addExecutionBlock方法添加任务1========{number = 3, name = (null)}
addExecutionBlock方法添加任务3========{number = 5, name = (null)}
addExecutionBlock方法添加任务2========{number = 4, name = (null)}
```

* 运用继承自NSOperation的子类

首先我们定义一个继承自NSOperation的类，然后重写它的main方法，之后就可以使用这个子类来进行相关的操作了。

```
/*******************"WHOperation.h"*************************/
 
#import @interface WHOperation : NSOperation
 
@end
 
 
/*******************"WHOperation.m"*************************/
 
#import "WHOperation.h"
 
@implementation WHOperation
 
- (void)main {
    for (int i = 0; i < 3; i++) {
        NSLog(@"NSOperation的子类WHOperation======%@",[NSThread currentThread]);
    }
}
 
@end
 
 
/*****************回到主控制器使用WHOperation**********************/
 
- (void)testWHOperation {
    WHOperation *operation = [[WHOperation alloc] init];
    [operation start];
}
```

运行结果如下，依然是在主线程执行。

```
SOperation的子类WHOperation======{number = 1, name = main}
NSOperation的子类WHOperation======{number = 1, name = main}
NSOperation的子类WHOperation======{number = 1, name = main}
```

所以，NSOperation是需要配合队列NSOperationQueue来实现多线程的。下面就来说一下队列NSOperationQueue。

### No.3：队列NSOperationQueue

NSOperationQueue只有两种队列：主队列、其他队列。其他队列包含了串行和并发。

主队列的创建如下，主队列上的任务是在主线程执行的。

```
NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
```

其他队列（非主队列）的创建如下，加入到‘非队列’中的任务默认就是并发，开启多线程。

```
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
```

注意：

非主队列（其他队列）可以实现串行或并行。
队列NSOperationQueue有一个参数叫做最大并发数：maxConcurrentOperationCount。
maxConcurrentOperationCount默认为-1，直接并发执行，所以加入到‘非队列’中的任务默认就是并发，开启多线程。
当maxConcurrentOperationCount为1时，则表示不开线程，也就是串行。
当maxConcurrentOperationCount大于1时，进行并发执行。
系统对最大并发数有一个限制，所以即使程序员把maxConcurrentOperationCount设置的很大，系统也会自动调整。所以把最大并发数设置的很大是没有意义的。

### No.4：NSOperation + NSOperationQueue

把任务加入队列，这才是NSOperation的常规使用方式。

* addOperation添加任务到队列
先创建好任务，然后运用- (void)addOperation:(NSOperation *)op 方法来吧任务添加到队列中，示例代码如下：

```
(void)testOperationQueue {
    // 创建队列，默认并发
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 创建操作，NSInvocationOperation
    NSInvocationOperation *invocationOperation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationOperationAddOperation) object:nil];
    // 创建操作，NSBlockOperation
    NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperation把任务添加到队列======%@", [NSThread currentThread]);
        }
    }];
 
    [queue addOperation:invocationOperation];
    [queue addOperation:blockOperation];
}
 
 
- (void)invocationOperationAddOperation {
    NSLog(@"invocationOperation===aaddOperation把任务添加到队列====%@", [NSThread currentThread]);
```

运行结果如下，可以看出，任务都是在子线程执行的，开启了新线程！

```
invocationOperation===addOperation把任务添加到队列===={number = 4, name = (null)}
addOperation把任务添加到队列======{number = 3, name = (null)}
addOperation把任务添加到队列======{number = 3, name = (null)}
addOperation把任务添加到队列======{number = 3, name = (null)}
```

* addOperationWithBlock添加任务到队列

这是一个更方便的把任务添加到队列的方法，直接把任务写在block中，添加到任务中。

```
- (void)testAddOperationWithBlock {
    // 创建队列，默认并发
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列======%@", [NSThread currentThread]);
        }
    }];
}
```

运行结果如下，任务确实是在子线程中执行。

```
addOperationWithBlock把任务添加到队列======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列======{number = 3, name = (null)}
```

* 运用最大并发数实现串行

上面已经说过，可以运用队列的属性maxConcurrentOperationCount（最大并发数）来实现串行，值需要把它设置为1就可以了，下面我们通过代码验证一下。

```
- (void)testMaxConcurrentOperationCount {
    // 创建队列，默认并发
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 最大并发数为1，串行
    queue.maxConcurrentOperationCount = 1;
 
    // 最大并发数为2，并发
//    queue.maxConcurrentOperationCount = 2;
 
 
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列1======%@", [NSThread currentThread]);
        }
    }];
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列2======%@", [NSThread currentThread]);
        }
    }];
 
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列3======%@", [NSThread currentThread]);
        }
    }];
}
```

运行结果如下，当最大并发数为1的时候，虽然开启了线程，但是任务是顺序执行的，所以实现了串行。

你可以尝试把上面的最大并发数变为2，会发现任务就变成了并发执行。

```
addOperationWithBlock把任务添加到队列1======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列1======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列1======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列2======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列2======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列2======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列3======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列3======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列3======{number = 3, name = (null)}
```

### No.5：NSOperation的其他操作

* 取消队列NSOperationQueue的所有操作，NSOperationQueue对象方法
```
- (void)cancelAllOperations
```
* 取消NSOperation的某个操作，NSOperation对象方法
```
- (void)cancel
```
* 使队列暂停或继续

```
// 暂停队列
[queue setSuspended:YES];
```
* 判断队列是否暂停
```
- (BOOL)isSuspended
```

暂停和取消不是立刻取消当前操作，而是等当前的操作执行完之后不再进行新的操作。

### No.6：NSOperation的操作依赖

NSOperation有一个非常好用的方法，就是操作依赖。可以从字面意思理解：某一个操作（operation2）依赖于另一个操作（operation1），只有当operation1执行完毕，才能执行operation2，这时，就是操作依赖大显身手的时候了

```
- (void)testAddDependency {
 
    // 并发队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 操作1
    NSBlockOperation *operation1 = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"operation1======%@", [NSThread  currentThread]);
        }
    }];
 
    // 操作2
    NSBlockOperation *operation2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"****operation2依赖于operation1，只有当operation1执行完毕，operation2才会执行****");
        for (int i = 0; i < 3; i++) {
            NSLog(@"operation2======%@", [NSThread  currentThread]);
        }
    }];
 
    // 使操作2依赖于操作1
    [operation2 addDependency:operation1];
    // 把操作加入队列
    [queue addOperation:operation1];
    [queue addOperation:operation2];
}
```

运行结果如下，操作2总是在操作1之后执行，成功验证了上面的说法。

```
operation1======{number = 3, name = (null)}
operation1======{number = 3, name = (null)}
operation1======{number = 3, name = (null)}
****operation2依赖于operation1，只有当operation1执行完毕，operation2才会执行****
operation2======{number = 4, name = (null)}
operation2======{number = 4, name = (null)}
operation2======{number = 4, name = (null)}
```

后记

* 本文所述的示例代码在这里：[WHMultiThreadDemo](https://github.com/remember17/WHMultiThreadDemo)
* 推荐简单又好用的分类集合：[WHKit](http://www.jianshu.com/p/c935314b078e)
* github地址：https://github.com/remember17




