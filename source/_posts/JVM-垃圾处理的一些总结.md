---
title: JVM 垃圾处理的一些总结
date: 2018-06-05 10:31:00
tags: java
id: 1528196487
---
## 内存管理
### 程序计数器
程序计数器（Program Counter Register），用来记录程序已经执行到的行号，当虚拟机在多个线程轮流切换时，靠它回到正确的执行位置。

### 堆和栈
Java 内存可以大致分为：分为虚拟机栈，本地方法栈，Java 堆，方法区，运行时常量池，直接内存。各自用来存储虚拟机和用户程序的代码、变量和其他一些信息。其中，Java 堆是所有线程共享的一块内存区域，唯一目的时存放对象实例。因此也成为垃圾管理的主要区域。

## 垃圾收集
### 判断对象是否需要被清理
判断对象是否需要被清理可以分为引用计数算法和可达性分析算法。

引用计数算法的原理是这样的：给对象中添加一个引用计数器，每当有一个地方引用它时，计数器就加1；当饮用失效时，计数器就减1；任何时刻计数器为0的对象就是不可能再被使用的。

可达性分析算法通过树来保存对象的引用链，如果对象和根（GC Roots）不再有引用链时，就被判断判断为不可达。此时对象不会被立即回收，而是要经历一个 finalize() 方法的回收过程，在此过程中对象可以自救。


### 垃圾收集算法
#### 标记-清除算法
最基础的收集算法是“标记-清除”（Mark-Sweep）算法，如同它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象，它的标记过程其实在前一节讲述对象标记判定时已经介绍过了。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其不足进行改进而得到的。它的主要不足有两个：一个是效率问题，标记和清除两个过程的效率都不高；另一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

#### 复制算法
为了解决效率问题，一种称为“复制”（Copying）的收集算法出现了，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。这样使得每次都是对整个半区进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为了原来的一半，未免太高了一点。

#### 标记-整理算法
复制收集算法在对象存活率较高时就要进行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

#### 分代收集算法
当前商业虚拟机的垃圾收集都采用“分代收集”（Generational Collection）算法，这种算法并没有什么新的思想，只是根据对象存活周期的不同将内存划分为几块。一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记—清理”或者“标记—整理”算法来进行回收。

--------------------------
参考资料：《深入理解Java虚拟机：JVM高级特性与最佳实践》