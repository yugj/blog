# 多线程

## 线程安全的实现方法
### 阻塞同步
* synchronized
  * 非公平锁，不可改实现
  * JVM实现
  * 不可中断
  * 优先使用synchronized
* ReentrantLock
  * 默认非公平锁，可改
  * JDK实现
  * 可中断
  * 需要自己控制编程场景使用

### 非阻塞同步
* CAS
  * 硬件支持原子性
* AtomicInteger
  * 发生冲突的做法是不断的进行重试
* ABA

### 无同步方案
* 栈封闭
  * 多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的
  * 说明：感觉没啥用
* Thread Local Storage
  * thread local map 存储线程上下文数据

