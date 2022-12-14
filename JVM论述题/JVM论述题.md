JVM论述题20220904



# JVM基本概念



## JVM运行时数据区相关概念

### 堆

堆，是JVM启动时划分的一块内存区域，用于存放对象，不同的JDK版本实现的方式不一样，不管是物理还是逻辑上主要分为：两代三区，即：新生代、老年代；新生代分为：eden和两个survivor， 是垃圾回收的主战场。

### 方法区

方法区，也是JVM各线程共享的一块内存区域，JVM启动时创建，关闭时释放，用于存放JVM加载的类型信息，运行时常量池、JIT代码缓存、Field信息、method信息等

### 运行时常量池（字符串常量池）

 1.7之前的版本在方法区中，之后的版本在堆中，存储字面型变量、引用类型的内存地址。

 字符串常量池，数据结构是StringTable,是一个类似KV结构的哈希表，存储用双引号标识的String以及都是常量相连接的字符串

### 虚拟机栈

虚拟机栈，JVM中每个线程都会创建各自的栈，每个栈默认大小为1m，可修改，栈的大小决定方法调用的深度，超出了会抛StackOverFlow异常。栈的基本元素，栈帧，存放局部变量表、操作数长，动态链接、方法返回地址等信息，另外，不需要GC。

### 本地方法栈

本地方法栈，与虚拟机栈作用相似，区别在于虚拟机栈为字节码服务，而本地方法栈是使用的Native服务。

### 直接内存

直接内存，不属于JVM内存结构，是物理机的内存，只是JVM可以调用该部分内存



## 为什么堆内存要分年轻代和老年代

主要基于弱分代和强分代假说，将对象根据存活的概率分类，把存活时间长的，放到固定区，从而减少扫描垃圾的时间，降低GC频率，提升JVM的性能。

# Java 对象的生命周期



## 对象的创建过程

 ![对象创建过程](https://github.com/alonly123/gk_java/blob/main/JVM%E8%AE%BA%E8%BF%B0%E9%A2%98/%E7%9B%B8%E5%85%B3%E5%9B%BE%E7%89%87/1662303252490.jpg)

 ## 对象的内存布局

  内存布局，包括： 对象头、数据区、对其填充

## 对象的销毁过程

    1. 先对eden 进行回收（MinorGC)，销毁无效对象；
    1. 将Eden中存活对象，放到S0
    1. 再把S0中的存活对象，复制到S1

## 对象的访问方式

 句柄、直接指针

## 为什么要内存担保

内存担保，是指老年代为新生代做担保，原因是新生代采用复制算法，当出现大量存活对象的时候，复制算法中S0和S1的空间都比较小，超出后，若不担保，就OOM了，所以需要老年代为其担保

# 垃圾收集器

## ParNew

​	参数：
​		XX:UseParNewGC
​		-XX:ParallelGCThreads=n
​			垃圾收集线程数
​	新生代并行-ParNew,老年代串行-SerialOld
​	Serial的多线程版本
​	单核CPU不建议使用

## ParallelScavenge

​	参数：
​		-XX:+UseParallelGC
​	特点
​		并行收集器
​		吞吐量优先，收集时需暂停线程
​			吞吐量=运行用户代码时间/(运行用户代码时间+运行垃圾收集时间)
​		新生代用并行-复制算法
​		老年代串行，标记-整理算法

## ParallelOld

​	参数：
​		-XX:+UseParallelOldGC
​	特点
​		并行收集器
​		吞吐量优先，收集时需暂停线程，对CPU敏感
​			吞吐量=运行用户代码时间/(运行用户代码时间+运行垃圾收集时间)
​		采用标记-整理算法

## CMS

​	参数
​		-XX:UseConcMarkSweepGC
​	特点
​		低延时，减少STW对用户的影响
​		并发收集，用户线程与收集线程一起执行，对CPU资源敏感
​		不会等待堆填满再收集，而是，到达阈值就开始收集
​		采用标记-清除算法，会产生内存碎片
​	过程
​		1。初始标记阶段
​			会STW，是为了标记出GCROOTS，可以关联的对象，关联对象较少，所以比较快
​		2。并发标记阶段
​			不会STW，遍历GCROOTS直接对象的引用链，耗时长
​		3。重新标记阶段
​			会STW，修正并发标记期间的新对象记录
​		4。并发清除阶段
​			不会STW，清除垃圾对象，释放内存空间

## G1

​	G1是面向服务端应用的全能型垃圾收集器，大内存企业配置主要是G1
​	参数：
​		-XX:UseG1GC
​	特点
​		吞吐量和低延时都行的整堆垃圾收集器
​		G1最大内存=32*2048=64G
​		G1最小堆内存=2G，低于此值不建议使用
​		全局使用标记-整理算法，局部使用复制算法
​		可预测的停顿，支持指定GC消耗时间，默认200ms
​	过程
​		1。初始标记阶段
​			会STW，是为了标记出GCROOTS，可以关联的对象，关联对象较少，所以比较快
​		2。并发标记阶段
​			不会STW，遍历GCROOTS直接对象的引用链，耗时长
​		3。最终标记阶段
​			会STW，修正并发阶段，标记产生变动的部分
​		4。 筛选回收
​			会STW，对各个Region的回收价值和成本排序，根据用户期望GC停顿时间确定回收计划
​	内存划分























