---
layout: post
title: "Java内存模型"
date: 2015-07-07 7:59
comments: true
external-url:
categories:
---

&nbsp;&nbsp;最近同事买了本书*七周七并发模型*, 我才知道七周七XX原来是个系列. 去年我曾经简单地翻看过*七周七语言*中的Io,Erlang, Prolog章节,书明显不是精通Java/C++大全那种事无巨细的风格, 内容更为浅显易懂,但各种不同语言背后设计思想的碰撞,对我却是极大的震撼.人是最容易囿于过去不思改变,越来越觉得大部分程序员,其实就是一台精密机器上的齿轮,只管做好自己的事情,很少能跳出来,看看其它部分其它人到底是怎么工作的, 这是个分工越来越细密的行业了.*七周七XX*系列就是一个简单易读的小梯子,踩着它,我们能越过分工和框架的小墙望见对岸那群忙忙碌碌的程序员到底是气宗的妖孽, 还是剑宗的魔徒.
&nbsp;&nbsp;*七周七并发*第一章讲了Thread & Lock, 对于第一天学习,布置了个作业,要求了解Java 内存模型,并有如下问题:
*  关于初始化安全性,Java内存模型给出了怎样的保证?(把英文术语翻译成中文, 总感觉理解起来很别扭, initialization safety比之初始化安全性,要容易理解得多,外文小说也是,翻译过来总是少了股中国味,牛排用豆瓣酱搭配烧烤,感觉奇奇怪怪)
*   什么是double check? 为什么说double check是反模式的?

&nbsp;&nbsp;关于Java内存模型的参考资料在[JSR 133(Java Memory Model)FAQ](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html#conclusion)中,内容主要以问答形式组织,想要深入了解的可以仔细读读.
##什么是内存模型?
&nbsp;&nbsp;大部分多处理器计算机都是采取单内存的结构,各个处理器通过总线竞争访问内存.为了加快内存访问速度,每个处理器都有各自的cache.cache带来一个内存可见性的问题,一个处理器对内存的写, 可能不是立即对其它处理器可见,一个处理器的读,也不一定是最新数据.在处理器层面上,内存模型定义了某处理器写入内存对其它处理器可见, 以及其它处理器写入内存对本处理器可见的充分必要条件.    
&nbsp;&nbsp;有些硬件中,采取的是强内存模型,即所有处理器所看见的任何内存值,都是最新的值;有些采取的是弱内存模型,需要一些特殊的指令(叫做内存屏障-memory barriers)将处理器cache中的值写入内存,才能让内存写入对其他处理器可见.    
&nbsp;&nbsp;直观上来说,强内存模型更易于编程,但是结果却恰恰相反.强内存模型在某些特殊情境下也需要内存屏障指令,目前处理器设计都是更鼓励弱内存模型,因为保持cache一致性的成本更低,在多处理器和大内存时,更易于扩展.    
&nbsp;&nbsp;对于内存模型来说,除了cache之外,还有一个重要的因素需要考虑-指令重排序.现今的CPU,为了加快指令执行速度,有各种各样的技术,例如流水线超流水线等等等,这些技术会打乱原本串行执行指令的顺序.在多线程时,其他线程看来,本线程有些指令的执行提前了.     
```java
Class Reordering {
  int x = 0, y = 0;
  public void writer() {
    x = 1;
    y = 2;
  }

  public void reader() {
    int r1 = y;
    int r2 = x;
  }
}
```
&nbsp;&nbsp;上述代码中,假如writer和reader在俩个线程中执行,对于reader线程来说,程序执行到r1 = 2时, 继续往下执行r2不一定等于1, 有可能是0.这是因为指令重排序的缘故,y = 2的指令可能在x = 1之前就执行了.    
&nbsp;&nbsp;Java内存模型描述了多线程代码中什么行为是合法的,以及线程间是如何通过内存交互.Java提供了内置语言中的volatile, final, synchronized关键字支持,内存模型定义了这些关键字的行为,并保证正确synchronized的Java程序在所有处理器架构上都能正确运行.
##除了Java,其他语言有内存模型么?
&nbsp;&nbsp;C/C++没有在语言内置对多线程的支持,因此也没有内存模型,对于多线程行为,更多地依赖于使用的库,编译器和平台.
##JSR内存模型的具体内容?
&nbsp;&nbsp;在1997年之后, 人们渐渐发现Java旧内存模型中一些违反人们直觉容易造成bug的部分,因此提出了JSR 133,新内存模型的不同有兴趣的可以仔细研究, 这里不细述.值得一提的是新内存模型添加了对初始化安全性的保证.如果一个对象是正确构建出来的(即在构建时并没有指向它的引用创造出来),那么所有线程中,如果有该对象的引用,该对象的所有final属性,线程所见的都是构建好的值, 无需使用synchronized关键字.
&nbsp;&nbsp;关于这点的例子可见Java String的实现,Java String对象内部使用三个属性保存状态, 一个是字符数组, 一个是offset:同数组头的偏离量,一个是字符串长度.这样的设计,使用String对象的子字符串可以和该对象共享内存,大大节省了内存使用.这个设计,肯定要求三个属性都是final的,否则子字符串轻易更改字符数组,一切都会乱套.在老的对象模型中, 由于初始化安全性没有保障,其它线程可能看见offset的值为0, 然后才变为4之类,一切都乱套了.
&nbsp;&nbsp;Java中的synchronized关键字,用于获取一个互斥锁.但synchronized关键字背后做的事情,不仅仅限于获取锁, 还有
*  当离开临界区,释放互斥锁时,会将cache中的值写入内存,因此此线程对内存的写入其他线程都立刻可见
*  获取锁后,进入临界区前,所有cache都会失效,变量的值都要存内存中重新读取.

&nbsp;&nbsp;对于volatile关键字,Java中保证所有对volatile属性读出的值都是最新的值,同时对于volatile属性的读写指令顺序也不受指令重排序影响.更进一步的说,非volatile属性的读写指令顺序也不一定受影响.对volatile属性的写入,会有释放锁时的效果,即所有cache的值会写入内存,而对volatile属性的读入,会让cache失效,从内存中读入值.如下代码显示了volatile的作用,感觉作用其实和锁差不多了.
```java
class VolatileExample {
  int x = 0;
  volatile boolean v = false;
  public void writer() {
    x = 42;
    v = true;
  }

  public void reader() {
    if (v == true) {
      //uses x - guaranteed to see 42.
    }
  }
}
```

&nbsp;&nbsp;Java内存模型在重排序的指令中保证了一个局部的有序.对于内存操作(读写一个变量, 获取锁, 释放锁), 和线程操作(线程start/join), 内存模型保证了某些操作确实如其在代码中顺序一样在其他操作之前发生(happen before), 即如果一个操作在代码中在其他操作之前,那么它会保证不受指令重排序影响,在其他操作之前运行,并且结果对其他操作可见.具体规则如下:
*   在同一个线程中, 每个操作都happen before在其后的操作
*   释放锁的操作肯定happen before获取锁操作
*   对volatile属性的写入happen before对该属性的读
*   对于线程来说, 这个线程的start()操作happen before所有线程内的执行代码
*   对于线程来说,所有join()后的代码, 必须等待join的线程执行完毕.


##什么是不正确的同步?
&nbsp;&nbsp;不正确是个抽象的词,具体来说,不正确的同步一般是有数据竞争.即
*  有一个线程写入某个变量
*  另外一个线程读入该变量
*  读写的顺序没有使用同步来协调

当你的代码中出现这种情况时,你就应该本能的感受到危险,采取措施了.

##double check在Java中有用么?
&nbsp;&nbsp;double check是实现lazy initialization时,为了避免同步成本的一个技巧.代码如下:
```java
// double-checked-locking - don't do this!

private static Something instance = null;

public Something getInstance() {
  if (instance == null) {
    synchronized (this) {
      if (instance == null)
        instance = new Something();
    }
  }
  return instance;
}
```
&nbsp;&nbsp;这个技巧看起来很聪明, 但是在java中,却没有用.具体原因可见:[Double-checked locking: Clever, but broken](http://www.javaworld.com/article/2074979/java-concurrency/double-checked-locking--clever--but-broken.html), 还是推荐使用Initialization On Demand Holder技巧,更为简单易懂, 也是线程安全的,代码如下.
```java
private static class LazySomethingHolder {
  public static Something something = new Something();
}

public static Something getInstance() {
  return LazySomethingHolder.something;
}
```
&nbsp;&nbsp;java中编译器对静态属性的初始化,会保证对所有县城是可见和正确的.

##为什么需要了解这些?
&nbsp;&nbsp;最大的理由就是多线程bug实在是太难复现和调试了,因此,与其出问题了再调试,不如学习足够多的知识,避免bug发生
