Oracle的HotSpot JVM   
JVM由4大部分组成：ClassLoader，Runtime Data Area，Execution Engine，Native Interface。  
Runtime Data Area则是存放数据的，分为五部分：Stack，Heap，Method Area，PC Register，Native Method Stack。

Method Area 取决于JVM的实现者
HotSpot JVM中把Method Area划分为非堆内存
NonHeap包含PermGen和Code Cache，PermGen包含Method Area,而且PermGen在JAVA SE 8中已经不再用了。
java8中PermGen已经从JVM中移除并被MetaSpace取代

栈帧:StackFrame包含三类信息：局部变量，执行环境，操作数栈

Heap:
Young Generation和Old Generation（也叫Tenured Generation）两大部分。Young Generation分为Eden和Survivor，Survivor又分为From Space和 ToSpace。


非堆内存(PermanentSpace 和 Code Cache)，和Heap一起组成JAVA内存 
Method Area包含Runtime Constant Pool（静态池）



栈一般会发生以下两种异常：

1.当线程中的计算所需要的栈超过所允许大小时，会抛出StackOverflowError。

2.当Java栈试图扩展时，没有足够的存储器来实现扩展，JVM会报OutOfMemoryError。    我针对栈进行了实验，由于递归的调用可以致使栈的引用增加，导致溢出，所以设计代码如下：

Old Space中则存放生命周期比较长的对象，而且有些比较大的新生对象也放在Old Space中。

堆的大小通过-Xms和-Xmx来指定最小值和最大值，
通过-Xmn来指定Young Generation的大小(Eden加FromSpace和ToSpace)
-XX:NewRatio来指定Eden区的大小



dump heap
GCFlag收集垃圾回收日志



1.标记清除算法，该算法是从根集合扫描整个空间，标记存活的对象，然后在扫描整个空间对没有被标记的对象进行回收，这种算法在存活对象较多时比较高效，但会产生内存碎片。

2.复制算法，该算法是从根集合扫描，并将存活的对象复制到新的空间，这种算法在存活对象少时比较高效。

3.标记整理算法，标记整理算法和标记清除算法一样都会扫描并标记存活对象，在回收未标记对象的同时会整理被标记的对象，解决了内存碎片的问题。


1.Serial GC。单线程的，所以它要求收集的时候所有的线程暂停。这对于高性能的应用是不合理的，所以串行GC一般用于Client模式的JVM中。

2.ParNew GC。是在SerialGC的基础上，增加了多线程机制。但是如果机器是单CPU的，这种收集器是比SerialGC效率低的。

3.Parrallel Scavenge GC。这种收集器又叫吞吐量优先收集器，而吞吐量=程序运行时间/(JVM执行回收的时间+程序运行时间),假设程序运行了100分钟，JVM的垃圾回收占用1分钟，那么吞吐量就是99%。Parallel Scavenge GC由于可以提供比较不错的吞吐量，所以被作为了server模式JVM的默认配置。

4.ParallelOld是老生代并行收集器的一种，使用了标记整理算法，是JDK1.6中引进的，在之前老生代只能使用串行回收收集器。

5.Serial Old是老生代client模式下的默认收集器，单线程执行，同时也作为CMS收集器失败后的备用收集器。

6.CMS又称响应时间优先回收器，使用标记清除算法。他的回收线程数为(CPU核心数+3)/4，所以当CPU核心数为2时比较高效些。CMS分为4个过程：初始标记、并发标记、重新标记、并发清除。

7.GarbageFirst（G1）。比较特殊的是G1回收器既可以回收Young Generation，也可以回收Tenured Generation。它是在JDK6的某个版本中才引入的，性能比较高，同时注意了吞吐量和响应时间。


以下几种情况会触发Full GC：

1.Tenured(Old) Space空间不足以创建打的对象或者数组，会执行FullGC，并且当FullGC之后空间如果还不够，那么会OOM:java heap space。

2.Permanet Generation的大小不足，存放了太多的类信息，在非CMS情况下回触发FullGC。如果之后空间还不够，会OOM:PermGen space。

3.CMS GC时出现promotion failed和concurrent mode failure时，也会触发FullGC。promotion failed是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的；concurrent mode failure是在执行CMS GC的过程中同时有对象要放入旧生代，而此时旧生代空间不足造成的。

4.判断MinorGC后，要晋升到TenuredSpace的对象大小大于TenuredSpace的大小，也会触发FullGC。





CMS XX:+UseConcMarkSweepGC  
CMS采用的基础算法是：标记—清除。默认会开启 -XX :+UseParNewGC，在年轻代使用并行复制收集。    
总得来说，CMS回收器减少了回收的停顿时间，但是降低了堆空间的利用率。
啥时候用CMS

如果你的应用程序对停顿比较敏感，并且在应用程序运行的时候可以提供更大的内存和更多的CPU(也就是硬件牛逼)，那么使用CMS来收集会给你带来好处。还有，如果在JVM中，有相对较多存活时间较长的对象(老年代比较大)会更适合使用CMS。


G1   
-XX:+UnlockExperimentalVMOptions -XX:+UseG1GC  
让应用达到准实时的效果，同时保持JVM堆空间的利用率，将作为CMS的替代者在JDK 7中闪亮登场

硬件环境的要求，必须是多核的CPU以及较大的内存（从规范来看，512M以上就满足条件了），另外一方面是需要接受吞吐量的稍微降低，对于实时性要求高的系统而言，这点应该是可以接受的。