# 线程池的创建方法
- https://www.cnblogs.com/pcheng/p/13540619.html
## 1.newCachedThreadPool
```java
// 创建一个可缓存的线程池,若线程数超过处理所需,缓存一段时间后会回收,若线程数不够,则新建线程。
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
// 创建一个固定大小的线程池,可控制并发的线程数,超出的线程会在队列中等待。
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
// (1)corePoolSize：核心线程数,线程池中始终存活的线程数。
// (2)maximumPoolSize: 最大线程数,线程池中允许的最大线程数。
// (3)keepAliveTime: 存活时间,线程没有任务执行时最多保持多久时间会终止。
// (4)unit: 单位,参数keepAliveTime的时间单位,7种可选。
// (5)workQueue: 一个阻塞队列,用来存储等待执行的任务,均为线程安全,7种可选。
// (6)threadFactory: 线程工厂,主要用来创建线程,默及正常优先级、非守护线程。
// (7)handler：拒绝策略,拒绝处理任务时的策略,4种可选,默认为AbortPolicy。
```

# 锁
- https://segmentfault.com/a/1190000023735772
## Synchronized
### 1.偏向锁
> 偏向锁是指一段同步代码一直被一个线程所访问,那么该线程会自动获取锁。降低获取锁的代价。
### 2.轻量级锁
> 轻量级锁是指当锁是偏向锁的时候,被另一个线程所访问,偏向锁就会升级为轻量级锁,其他线程会通过自旋的形式尝试获取锁,不会阻塞,提高性能。默认10次
### 3.重量级锁
> 重量级锁是指当锁为轻量级锁的时候,另一个线程虽然是自旋,但自旋不会一直持续下去,当自旋一定次数的时候,还没有获取到锁,就会进入阻塞,该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞,性能降低。

### 公平锁,非公平锁
> 当一个线程持有的锁释放时,其他线程按照先后顺序,先申请的先得到锁,那么这个锁就是公平锁。反之,如果后申请的线程有可能先获取到锁,就是非公平锁。

### 乐观锁,悲观锁
> 悲观锁:即在读数据的时候总认为其他线程会对数据进行修改,所以采取加锁的形式,一旦本线程要读取数据时,就加锁,其他线程被阻塞,等待锁的释放。所以悲观锁总结为悲观加锁阻塞线程。
> 乐观锁实际上是没有锁的,只是通过一种比较交换的方法来保证数据同步,总结为乐观无锁回滚重试。

### 可重入锁
> 可重入锁即允许多个线程多次获取同一把锁,那从锁本身的角度来看,就是可以重新进入该锁。比如有一个递归函数里面有加锁操作,如果这个锁不阻塞自己,就是可重入锁,故也称递归锁。

### 可中断锁
> 如果线程A持有锁,线程B等待获取该锁。由于线程A持有锁的时间过长,线程B不想继续等待了,我们可以让线程B中断自己或者在别的线程里中断它,这种就是可中断锁。
> 在 Java 中,synchronized就是不可中断锁,而Lock的实现类都是可中断锁。

### 独享锁,共享锁
> 独享锁亦称互斥锁,排它锁,容易理解这种锁每次只允许一个线程持有。反之,就是共享锁啦。互斥锁/读写锁就是具体的实现。

### 读锁,写锁
> 读写锁就是其最典型的锁。写锁是独享锁,读锁是共享锁。

## synchronized 与 ReentrantLock 的区别
> 如果使用 synchronized,如果A不释放,B将一直等下去,不能被中断
> 如果使用 ReentrantLock,如果A不释放,可以使B在等待了足够长的时间以后,中断等待,而干别的事情
> 在资源竞争不是很激烈的情况下,Synchronized 的性能要优于 ReetrantLock,但是在资源竞争很激烈的情况下,Synchronized 的性能会下降几十倍,但是 ReetrantLock 的性能能维持常态

## synchronized 与 Lock 的区别
> synchronized是在JVM层面上实现的,不但可以通过一些监控工具监控synchronized的锁定,而且在代码执行时出现异常,JVM会自动释放锁定,但是使用Lock则不行,lock是通过代码实现的,要保证锁定一定会被释放,就必须将unLock()放到finally{}中