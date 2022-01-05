# 集合
- https://blog.csdn.net/feiyanaffection/article/details/81394745

# 设计模式
- http://c.biancheng.net/design_pattern/
> 它是解决特定问题的一系列套路,是前辈们的代码设计经验的总结,具有一定的普遍性,可以反复使用
- 1.开闭原则: 当对扩展开放,对修改关闭
- 2.里氏替换原则: 继承必须确保超类所拥有的性质在子类中仍然成立
- 3.依赖倒置原则: 高层模块不应该依赖低层模块,两者都应该依赖其抽象;抽象不应该依赖细节,细节应该依赖抽象
- 4.单一职责原则: 一个类应该有且仅有一个引起它变化的原因,否则类应该被拆分
- 5.接口隔离原则: 一个类对另一个类的依赖应该建立在最小的接口上
- 6.迪米特法则: 如果两个软件实体无须直接通信,那么就不应当发生直接的相互调用,可以通过第三方转发该调用 其目的是降低类之间的耦合度,提高模块的相对独立性 
- 7.合成复用原则: 尽量先使用组合或者聚合等关联关系来实现,其次才考虑使用继承关系来实现 

# 线程的创建方式
- 1.实现 Runnable 接口
- 2.继承 Thread 类,Thread 类也是 Runnable 接口的子类,但在Thread类中并没有完全实现 Runnable 接口中的 run() 方法
> 如果一个类继承 Thread类,则不适合于多个线程共享资源,而实现了 Runnable 接口,就可以方便的实现资源的共享

# 线程的状态
- 1.创建状态 
> 在程序中用构造方法创建了一个线程对象后,新的线程对象便处于新建状态,此时它已经有了相应的内存空间和其他资源,但还处于不可运行状态 新建一个线程对象可采用Thread 类的构造方法来实现,例如 "Thread thread=new Thread()" 
- 2.就绪状态 
> 新建线程对象后,调用该线程的 start() 方法就可以启动线程 当线程启动时,线程进入就绪状态 此时,线程将进入线程队列排队,等待 CPU 服务,这表明它已经具备了运行条件 
- 3.运行状态 
> 当就绪状态被调用并获得处理器资源时,线程就进入了运行状态 此时,自动调用该线程对象的 run() 方法 run() 方法定义该线程的操作和功能 
- 4.阻塞状态 
> 一个正在执行的线程在某些特殊情况下,如被人为挂起或需要执行耗时的输入/输出操作,会让 CPU 暂时中止自己的执行,进入阻塞状态 在可执行状态下,如果调用sleep(),suspend(),wait() 等方法,线程都将进入阻塞状态,发生阻塞时线程不能进入排队队列,只有当引起阻塞的原因被消除后,线程才可以转入就绪状态 
- 5.死亡状态 
> 线程调用 stop() 方法时或 run() 方法执行结束后,即处于死亡状态 处于死亡状态的线程不具有继续运行的能力 
>> 备注: Java 程序每次运行至少启动几个线程？2个,main,垃圾回收线程

# JVM 内存模型
## 类加载器 -> 执行引擎 -> 运行时数据区(内存)
* 1.程序计数器: 程序计数器是一块很小的内存空间,它是线程私有的,可以认作为当前线程的行号指示器 
> 一条线程中有多个指令,为了线程切换可以恢复到正确执行位置,每个线程都需有独立的一个程序计数器,不同线程之间的程序计数器互不影响,独立存储
* 2.Java栈(虚拟机栈): 栈描述的是Java方法执行的内存模型,存储局部变量
> 每一个方法被调用的过程就对应一个栈帧在虚拟机栈中从入栈到出栈的过程,StackOverflowError,无法扩展时 OutOfMemory
    - 虚拟机栈: 执行Java方法
    - 本地方法栈: native 方法,底层调用的c或者c++
* 3.堆: 所有线程共享的,它的目的是存放对象实例,同时它也是GC所管理的主要区域,因此常被称为GC堆
    - 新生代: 
    - 老年代: 
    - 永久代: 
    - -Xmx,-Xms
* 4.方法区: 同堆一样,是所有线程共享的内存区域,为了区分堆,又被称为非堆
> 用于存储已被虚拟机加载的类信息、常量、静态变量,如static修饰的变量
* 确定哪些是垃圾的算法
    - 1.引用计数算法: 引用+1,引用删除-1,0删除对象
    - 2.可达性分析算法: GC ROOT(2个栈,方法区中引用对的对象) -> 向下搜索,没有与其关联的,可以删除
* 垃圾回收算法
    - 1.标记清除算法: 把标记的删除,有内存碎片
    - 2.复制算法: 2个区域,存活的复制放1个,然后删除另一个,代价高,大内存被分为2块小内存
    - 3.标记整理算法: 让存活的对象移动到一端,清理另一端边界,内存变动频繁
    - 4.分代收集算法: 新生代采用复制算法,老年代采用标记清除,标记整理算法

# 锁
- https://segmentfault.com/a/1190000023735772

## Synchronized
### 1.偏向锁
> 偏向锁是指一段同步代码一直被一个线程所访问,那么该线程会自动获取锁 降低获取锁的代价 
### 2.轻量级锁
> 轻量级锁是指当锁是偏向锁的时候,被另一个线程所访问,偏向锁就会升级为轻量级锁,其他线程会通过自旋的形式尝试获取锁,不会阻塞,提高性能 默认10次
### 3.重量级锁
> 重量级锁是指当锁为轻量级锁的时候,另一个线程虽然是自旋,但自旋不会一直持续下去,当自旋一定次数的时候,还没有获取到锁,就会进入阻塞,该锁膨胀为重量级锁 重量级锁会让其他申请的线程进入阻塞,性能降低 
### 4.公平锁,非公平锁
> 当一个线程持有的锁释放时,其他线程按照先后顺序,先申请的先得到锁,那么这个锁就是公平锁 反之,如果后申请的线程有可能先获取到锁,就是非公平锁 
### 5.乐观锁,悲观锁
- 悲观锁即在读数据的时候总认为其他线程会对数据进行修改,所以采取加锁的形式,一旦本线程要读取数据时,就加锁,其他线程被阻塞,等待锁的释放 所以悲观锁总结为悲观加锁阻塞线程 
- 乐观锁实际上是没有锁的,只是通过一种比较交换的方法来保证数据同步,总结为乐观无锁回滚重试 
### 6.可重入锁
> 可重入锁即允许多个线程多次获取同一把锁,那从锁本身的角度来看,就是可以重新进入该锁 比如有一个递归函数里面有加锁操作,如果这个锁不阻塞自己,就是可重入锁,故也称递归锁 
### 7.可中断锁
> 如果线程A持有锁,线程B等待获取该锁 由于线程A持有锁的时间过长,线程B不想继续等待了,我们可以让线程B中断自己或者在别的线程里中断它,这种就是可中断锁 
> 在 Java 中,synchronized就是不可中断锁,而Lock的实现类都是可中断锁 
### 8.独享锁,共享锁
> 独享锁亦称互斥锁,排它锁,容易理解这种锁每次只允许一个线程持有 反之,就是共享锁啦 互斥锁/读写锁就是具体的实现 
### 9.读锁,写锁
> 读写锁就是其最典型的锁 写锁是独享锁,读锁是共享锁 

## synchronized 与 ReentrantLock 的区别
* 如果使用 synchronized,如果A不释放,B将一直等下去,不能被中断
* 如果使用 ReentrantLock,如果A不释放,可以使B在等待了足够长的时间以后,中断等待,而干别的事情
* 在资源竞争不是很激烈的情况下,Synchronized 的性能要优于 ReetrantLock,但是在资源竞争很激烈的情况下,Synchronized 的性能会下降几十倍,但是 ReetrantLock 的性能能维持常态

## synchronized 与 Lock 的区别
> synchronized是在JVM层面上实现的,不但可以通过一些监控工具监控synchronized的锁定,而且在代码执行时出现异常,JVM会自动释放锁定,但是使用Lock则不行,lock是通过代码实现的,要保证锁定一定会被释放,就必须将unLock()放到finally{}中

# 线程池的创建方法
- https://www.cnblogs.com/pcheng/p/13540619.html
* 1.newCachedThreadPool: 创建一个可缓存的线程池,若线程数超过处理所需,缓存一段时间后会回收,若线程数不够,则新建线程 
* 2.newFixedThreadPool: 创建一个固定大小的线程池,可控制并发的线程数,超出的线程会在队列中等待 
* 3.newScheduledThreadPool: 创建一个周期性的线程池,支持定时及周期性执行任务
* 4.newSingleThreadExecutor: 创建一个单线程的线程池,可保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行
* 5.ThreadPoolExecutor: ThreadPoolExecutor类提供了4种构造方法,可根据需要来自定义一个线程池

## 1.newCachedThreadPool
```java
// 创建一个可缓存的线程池,若线程数超过处理所需,缓存一段时间后会回收,若线程数不够,则新建线程 
private static void createCachedThreadPool() {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 10; i++) {
        final int index = i;
        executorService.execute(() -> {
            // 获取线程名称,默认格式:pool-1-thread-1
            System.out.println(DateUtil.now() + " " + Thread.currentThread().getName() + " " + index);
            // 等待2秒
            sleep(2000);
       });
   }
}
```
## 2.newFixedThreadPool
```java
// 创建一个固定大小的线程池,可控制并发的线程数,超出的线程会在队列中等待 
private static void createFixedThreadPool() {
    ExecutorService executorService = Executors.newFixedThreadPool(3);
    for (int i = 0; i < 10; i++) {
        final int index = i;
        executorService.execute(() -> {
            // 获取线程名称,默认格式:pool-1-thread-1
            System.out.println(DateUtil.now() + " " + Thread.currentThread().getName() + " " + index);
            // 等待2秒
            sleep(2000);
       });
   }
}
```
## 3.newScheduledThreadPool
```java
// 创建一个周期性的线程池,支持定时及周期性执行任务
private static void createScheduledThreadPool() {
    ScheduledExecutorService executorService = Executors.newScheduledThreadPool(3);
    System.out.println(DateUtil.now() + " 提交任务");
    for (int i = 0; i < 10; i++) {
        final int index = i;
        executorService.schedule(() -> {
            // 获取线程名称,默认格式:pool-1-thread-1
            System.out.println(DateUtil.now() + " " + Thread.currentThread().getName() + " " + index);
            // 等待2秒
           sleep(2000);
       }, 3, TimeUnit.SECONDS);
   }
}
```
## 4.newSingleThreadExecutor
```java
// 创建一个单线程的线程池,可保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行
private static void createSingleThreadPool() {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    for (int i = 0; i < 10; i++) {
        final int index = i;
        executorService.execute(() -> {
            // 获取线程名称,默认格式:pool-1-thread-1
            System.out.println(DateUtil.now() + " " + Thread.currentThread().getName() + " " + index);
            // 等待2秒
            sleep(2000);
       });
   }
}
```
## 5.ThreadPoolExecutor
```java
// ThreadPoolExecutor类提供了4种构造方法,可根据需要来自定义一个线程池
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        // 省略...
    }
// (1)corePoolSize：核心线程数,线程池中始终存活的线程数 
// (2)maximumPoolSize: 最大线程数,线程池中允许的最大线程数 
// (3)keepAliveTime: 存活时间,线程没有任务执行时最多保持多久时间会终止 
// (4)unit: 单位,参数keepAliveTime的时间单位,7种可选 
// (5)workQueue: 一个阻塞队列,用来存储等待执行的任务,均为线程安全,7种可选 
// (6)threadFactory: 线程工厂,主要用来创建线程,默及正常优先级、非守护线程 
// (7)handler：拒绝策略,拒绝处理任务时的策略,4种可选,默认为AbortPolicy 
```
