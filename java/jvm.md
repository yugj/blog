# JVM

## Java程序执行过程
.java -> .class -> 类加载器 -> 字节码校验器(-> 解释器/jit代码生成器) -> 硬件

## Java虚拟机结构
![](/assets/jvm.webp)

### 方法区（栈/非堆）
存储类信息、常量池、静态变量、JIT编译后的代码等数据

### 堆
* 新生代 新创建对象和未GC过的对象
  * 伊甸园 8
    * TLAB JVM 为每个线程分配了一个私有缓存区域，它包含在 Eden 空间内
    * 使用 TLAB 可以避免一系列的非线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为快速分配策略
    * 若程序需要TLAB较多可，-XX:TLABWasteTargetPercent 配置大小，默认1%
  * 幸存区 1
  * 幸存区 1
* 老年代 经过一定GC后转入老年代，或大对象（避免在新生代区域拷贝）
* 持久代(java8 改元空间) 
  * 永久代和元空间都可以理解为方法区的落地实现
  * 永久代物理是堆的一部分，和新生代，老年代地址是连续的（受垃圾回收器管理），而元空间存在于本地内存（我们常说的堆外内存，不受垃圾回收器管理），这样就不受 JVM 限制了，也比较难发生OOM（都会有溢出异常）
    
### 程序计数器
PC 寄存器用来存储指向下一条指令的地址，即将要执行的指令代码。由执行引擎读取下一条指令。  
JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。

### 虚拟机栈
```
每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame），对应着一次次 Java 方法调用，是线程私有的，生命周期和线程一致。
可以简单理解为线程工作栈，xss设置线程的最大栈空间
```
作用：主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。


### 本地方法区
```
简单的讲，一个 Native Method 就是一个 Java 调用非 Java 代码的接口。我们知道的 Unsafe 类就有很多本地方法。
```

## 特性&优化建议
* 堆是JVM中所有线程共享,在其上进行对象内存的分配均需要进行加锁
  * -> new开销大，减少不必要的new操作，特别是网关相关大流量入口
* 所有线程有独立的空间TLAB（Thread Local Allocation Buffer），不需要加锁，对象过大还是会走队内存
* 新创建对象在Yong Generation，经过一次或多次GC后转入OldGeneration
* 方法区对应PermanetGeneration（持久代），存放类的信息（名称、修饰符等）、类中的静态变量、类中定义为final类型的常量、类中的Field信息、类中的方法信息，也是全局共享,当方法区域需要使用的内存超过其允许的大小时，会抛出OutOfMemory的错误信息。
  * -> 

## OutOfMemory 和 StackOverFlow
* OutOfMemoryError is related to Heap.非堆内存满，例如netty 可能涉及非堆内存访问
* StackOverflowError is related to stack，对内存满，大量创建堆内存导致

## 对象是否存活判断
1.）引用计数算法：给对象中添加一个引用计数器，每当一个地方应用了对象，计数器加1；当引用失效，计数器减1；当计数器为0表示该对象已死、可回收。但是它很难解决两个对象之间相互循环引用的情况。  
2.）可达性分析算法：通过一系列称为“GC Roots”的对象作为起点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连（即对象到GC Roots不可达），则证明此对象已死、可回收。Java中可以作为GC Roots的对象包括：虚拟机栈中引用的对象、本地方法栈中Native方法引用的对象、方法区静态属性引用的对象、方法区常量引用的对  
在主流的商用程序语言（如我们的Java）的主流实现中，都是通过可达性分析算法来判定对象是否存活的。

## 垃圾回收
### CMS
**四个流程**
* 可达性分析
* 标记清除
  * 初始标记: 仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿
  * 并发标记: 进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
  * 重新标记: 为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
  * 并发清除: 不需要停顿。

**缺点**
* 吞吐量低: 低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。 
* 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。 
* 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC

### G1
G1(Garbage-First)，它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。   
堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 G1 可以直接对新生代和老年代一起回收  

G1 把堆划分成多个大小相等的独立区域(Region)，新生代和老年代不再物理隔离，将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收，这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间(这两个值是通过过去回收的经验获得)，并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region
  

## 参考
* jvm https://zhuanlan.zhihu.com/p/91274291
* OutOfMemory VS StackOverFlow https://stackoverflow.com/questions/11435613/whats-the-difference-between-stackoverflowerror-and-outofmemoryerror
* 元空间和永久代 https://juejin.cn/post/6844904020964802574
* jvm https://pdai.tech/md/java/jvm/java-jvm-struct.html#11-%E4%BD%9C%E7%94%A8
* jvm gc https://pdai.tech/md/java/jvm/java-jvm-gc.html
