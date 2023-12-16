---
title: JUC
date: 2021-09-05 19:06:29
tags: JUC多线程
---
# 线程创建的方式
1.继承Thread类  
2.实现Runnable接口  
3.实现Callable接口
## Thread
- 自定义线程类继承Thread类
- 重写run()方法，编写线程执行体
- 创建线程对象，调用start()方法启动线程
## Runnable
- 实现Runnable接口，重写run方法
- 创建Runnable接口的实现类对象
- 创建线程类对象，通过线程对象来开启线程  

**发现问题：多个线程操作同一个资源的情况下，线程不安全，数据紊乱**
## 方法
- setPriority:更改线程的优先级
- sleep:在指定的毫秒数内让线程休眠
- join:等待该线程终止
- yield:暂停当前正在执行的线程，执行其他线程(礼让不一定成功)
- interrupt:中断线程
- isAlive:测试线程是否处于激活状态
## Callable
- 实现call方法
- 通过FutureTask进行开启线程
# 锁
## synchronized (关键字)
```java
/**
 * user:lufei
 * DATE:2021/9/5
 **/
public class Ticket {
    public static void main(String[] args) {

        TicketSale ticket = new TicketSale();
        new Thread(()->{
            for (int i = 0; i < 20; i++) {
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 20; i++) {
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 20; i++) {
                ticket.sale();
            }
        },"C").start();
    }
}
class TicketSale{
    private int number = 60;
    //本质：队列，锁
    public synchronized void sale(){
        if(number>0){
            System.out.println(Thread.currentThread().getName()+"卖出了第"+(number--)+"张票");
        }
    }
}
```
> 输出结果
```test
A卖出了第60张票
A卖出了第59张票
A卖出了第58张票
A卖出了第57张票
A卖出了第56张票
A卖出了第55张票
A卖出了第54张票
A卖出了第53张票
A卖出了第52张票
A卖出了第51张票
A卖出了第50张票
A卖出了第49张票
A卖出了第48张票
C卖出了第47张票
C卖出了第46张票
C卖出了第45张票
C卖出了第44张票
C卖出了第43张票
C卖出了第42张票
C卖出了第41张票
C卖出了第40张票
C卖出了第39张票
C卖出了第38张票
C卖出了第37张票
C卖出了第36张票
C卖出了第35张票
C卖出了第34张票
C卖出了第33张票
C卖出了第32张票
C卖出了第31张票
C卖出了第30张票
C卖出了第29张票
C卖出了第28张票
B卖出了第27张票
B卖出了第26张票
B卖出了第25张票
B卖出了第24张票
B卖出了第23张票
B卖出了第22张票
B卖出了第21张票
B卖出了第20张票
B卖出了第19张票
B卖出了第18张票
B卖出了第17张票
B卖出了第16张票
A卖出了第15张票
A卖出了第14张票
A卖出了第13张票
A卖出了第12张票
A卖出了第11张票
A卖出了第10张票
A卖出了第9张票
B卖出了第8张票
B卖出了第7张票
B卖出了第6张票
B卖出了第5张票
B卖出了第4张票
B卖出了第3张票
B卖出了第2张票
B卖出了第1张票
```
## Lock (接口)
```java
/**
 * user:lufei
 * DATE:2021/9/5
 **/
public class Ticket2 {
    public static void main(String[] args) {
        TicketSale2 ticket = new TicketSale2();
        new Thread(()->{for (int i = 0; i < 20; i++) ticket.sale();},"A").start();
        new Thread(()->{for (int i = 0; i < 20; i++) ticket.sale();},"B").start();
        new Thread(()->{for (int i = 0; i < 20; i++) ticket.sale();},"C").start();
    }
}
class TicketSale2{
    private int number = 60;
    Lock lock = new ReentrantLock();
    public void sale(){
        lock.lock();
        try {
            if(number>0){
                System.out.println(Thread.currentThread().getName()+"卖出了第"+(number--)+"张票");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```
> 输出结果
```text
A卖出了第60张票
A卖出了第59张票
A卖出了第58张票
A卖出了第57张票
A卖出了第56张票
A卖出了第55张票
A卖出了第54张票
A卖出了第53张票
A卖出了第52张票
A卖出了第51张票
A卖出了第50张票
A卖出了第49张票
A卖出了第48张票
A卖出了第47张票
A卖出了第46张票
A卖出了第45张票
A卖出了第44张票
A卖出了第43张票
A卖出了第42张票
A卖出了第41张票
B卖出了第40张票
B卖出了第39张票
B卖出了第38张票
B卖出了第37张票
B卖出了第36张票
B卖出了第35张票
B卖出了第34张票
B卖出了第33张票
B卖出了第32张票
B卖出了第31张票
B卖出了第30张票
B卖出了第29张票
B卖出了第28张票
B卖出了第27张票
B卖出了第26张票
B卖出了第25张票
B卖出了第24张票
B卖出了第23张票
B卖出了第22张票
B卖出了第21张票
C卖出了第20张票
C卖出了第19张票
C卖出了第18张票
C卖出了第17张票
C卖出了第16张票
C卖出了第15张票
C卖出了第14张票
C卖出了第13张票
C卖出了第12张票
C卖出了第11张票
C卖出了第10张票
C卖出了第9张票
C卖出了第8张票
C卖出了第7张票
C卖出了第6张票
C卖出了第5张票
C卖出了第4张票
C卖出了第3张票
C卖出了第2张票
C卖出了第1张票
```
## Lock和synchronized的区别
- synchronized 为内置关键字，而Lock为一个接口
- synchronized (自动的)无法判断获取锁的状态，Lock可以判断是否获取到了锁
- synchronized 会自动释放锁，Lock手动释放锁，如果不释放锁，死锁。
- synchronized 如果一个线程获得锁，另一个线程会一直等待，Lock则不一定会一直等待
- synchronized 可重入锁，不可以中断的，非公平锁，Lock可重入，可以判断锁，非公平(可以自己设置)。
- synchronized 适合锁少量的代码，Lock适合锁大量的代码。

# 常用辅助类
- CountDownLatch
```java
public class test1 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(7);    //new的时候给一个值，执行时将其减为0

        for(int i = 1;i<=7;i++){
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集到了");       //每执行一条线程，值减1，
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }

        countDownLatch.await();           //当值不为0时阻塞等待

        System.out.println("召唤神龙");
    }
}
```
```text
4收集到了
5收集到了
3收集到了
1收集到了
7收集到了
2收集到了
6收集到了
召唤神龙
```
- CyclicBarrier
```java
public class CyclicBarrierTest {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{     //与countDownLatch相反，每执行一条线程，值加1，当值为7时执行操作
            System.out.println("召唤龙神");
        });

        for (int i=1;i<=7;i++){
            final int temp = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集了"+temp+"颗龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```
```text
Thread-4收集了5颗龙珠
Thread-1收集了2颗龙珠
Thread-5收集了6颗龙珠
Thread-0收集了1颗龙珠
Thread-3收集了4颗龙珠
Thread-2收集了3颗龙珠
Thread-6收集了7颗龙珠
召唤龙神
```
- Semaphore
```java
public class SemaphoreTest {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);          //创建一定数量的许可，线程得到许可执行，否则阻塞等待
        for (int i=1;i<=6;i++){
            new Thread(()->{
                try {
                    semaphore.acquire();  //得到许可
                    System.out.println(Thread.currentThread().getName()+"得到了车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"走了");

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();   //释放       执行完之后释放许可
                }
            },String.valueOf(i)).start();
        }
    }
}
```
```text
2得到了车位
1得到了车位
3得到了车位
1走了
2走了
6得到了车位
3走了
4得到了车位
5得到了车位
4走了
5走了
6走了
```
# 读写锁ReadWriteLock
- 读锁：可以共享，可以同时多个人读
- 写锁：不能共享，同时只能有一个人写
```java
public class ReadWriteLockTest {
    public static void main(String[] args) {
        myCache cache = new myCache();

        //开启五个线程写
        for(int i=1;i<=5;i++){
            final int temp = i;
            new Thread(()->{
                cache.put(temp,temp);
            },String.valueOf(i)).start();
        }

        //开启五个线程读
        for(int i = 1;i<=5;i++){
            final int temp = i;
            new Thread(()->{
                cache.get(temp);
            },String.valueOf(i)).start();
        }

    }
}

class myCache{
    private final Map<Object,Object> map = new HashMap<>();
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(Object o1,Object o2){
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"写入"+o1);
            map.put(o1,o2);
            System.out.println(Thread.currentThread().getName()+"写入ok");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            readWriteLock.writeLock().unlock();
        }
    }

    public void get(Object o1){

        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"读取"+o1);
            map.get(o1);
            System.out.println(Thread.currentThread().getName()+"读取成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            readWriteLock.readLock().unlock();
        }

    }
}
```
写操作具有原子性，不能被插入，读可以同时多个读。
```text
3写入3
3写入ok
5写入5
5写入ok
4写入4
4写入ok
1写入1
1写入ok
2写入2
2写入ok
2读取2
5读取5
4读取4
1读取1
3读取3
3读取成功
4读取成功
2读取成功
1读取成功
5读取成功
```
# 阻塞队列(BlockingQueue)
在collection下，与List,Set同级。
使用情况：多线程并发处理，线程池！

捕捉错误无返回值  使用 add   remove  
有返回值的使用   offer  poll  
阻塞等待使用     put   take  
超时等待使用     offer  poll

>SynchronousQueue 同步队列

没有容量，进去一个元素，必须等待取出来，才能再放进一个元素。

put  take
# 线程池
## 三大方法
```java
public class Demo01 {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();//单个
        //ExecutorService executorService = Executors.newFixedThreadPool(5);   //创建固定大小的线程池
        //ExecutorService executorService = Executors.newCachedThreadPool();   //可变大小

        try {
            for(int i=1;i<=10;i++){
                executorService.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //线程池用完，程序结束，关闭线程池
            executorService.shutdown();
        }
    }
}
```
单一线程：
```text
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok
pool-1-thread-1ok

Process finished with exit code 0
```
固定5个线程：
```text
pool-1-thread-2ok
pool-1-thread-4ok
pool-1-thread-5ok
pool-1-thread-1ok
pool-1-thread-3ok
pool-1-thread-1ok
pool-1-thread-5ok
pool-1-thread-2ok
pool-1-thread-4ok
pool-1-thread-3ok

Process finished with exit code 0
```
可变大小：
```text
pool-1-thread-5ok
pool-1-thread-8ok
pool-1-thread-3ok
pool-1-thread-9ok
pool-1-thread-7ok
pool-1-thread-6ok
pool-1-thread-1ok
pool-1-thread-2ok
pool-1-thread-10ok
pool-1-thread-4ok

Process finished with exit code 0
```
单个源码：
```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,newLinkedBlockingQueue<Runnable>()));
}
```
固定个数源码：
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
    }
```
可变大小源码：
```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,   //  21亿   OOM
        60L, TimeUnit.SECONDS,
        new SynchronousQueue<Runnable>());
    }
```
## 七大参数
本质：ThreadPoolExecutor()
```java
public ThreadPoolExecutor(int corePoolSize,    //核心线程池大小
                              int maximumPoolSize,    //最大核心线程池大小
                              long keepAliveTime,     //超时了没人调用会释放
                              TimeUnit unit,      //超时单位
                              BlockingQueue<Runnable> workQueue,     //阻塞队列
                              ThreadFactory threadFactory,       //线程工厂 。创建线程，一般不用动
                              RejectedExecutionHandler handler    //拒绝策略
                              ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```
> 手动创建线程池
```java
public class Demo01 {
    public static void main(String[] args) {
        ThreadPoolExecutor executorService = new ThreadPoolExecutor(2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy());   //队列满了，尝试去和最早的竞争，成功执行，失败就无了，不抛出异常

        try {
            for(int i=1;i<=10;i++){
                executorService.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //线程池用完，程序结束，关闭线程池
            executorService.shutdown();
        }
    }
}
```
## 四大拒绝策略
```java
new ThreadPoolExecutor.AbortPolicy()    //满了，还有线程，不进行处理抛出异常
new ThreadPoolExecutor.CallerRunsPolicy()    //哪来回哪去
new ThreadPoolExecutor.DiscardPolicy()   //队列满了，丢掉任务，不会抛出异常
new ThreadPoolExecutor.DiscardOldestPolicy()   //队列满了，尝试去和最早的竞争，成功执行，失败就无了，不抛出异常
```
## 小结
线程池最大数量到底该如何定义
- cpu密集型，几核就定义为几，可以保证效率最高，
```java
Runtime.getRuntime().availableProcessors()   //获取cpu核数
```
- IO密集型，判断程序中十分耗资源的IO的线程数量，大于这个数量即可，一般可以设置两倍
# 四大函数式接口
lambda表达式，链式编程，函数式接口，Stream流式计算
>函数式接口:只有一个方法的接口
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
函数型接口测试：Function<T,R>   T为传入参数类型，R为返回结果类型
```java
public class test {
    public static void main(String[] args) {
        /*Function function = new Function<String,String>() {
            @Override
            public String apply(String o) {
                return o;
            }
        };
        System.out.println(function.apply("sdf"));*/
        Function<String,String> function = (o)->{return o;};
        System.out.println(function.apply("sdf"));
    }
}
```
断定型接口测试：Predicate<T> T为传入参数类型
```java
/*
 *  断定型接口：有一个输入参数返回值，只能是boolean值
 * */
public class test2 {
    public static void main(String[] args) {
        /*Predicate<String> stringPredicate = new Predicate<>(){

            @Override
            public boolean test(String s) {
                return s=="asdf";
            }
        };
        System.out.println(stringPredicate.test("asdf"));*/
        Predicate<String> predicate = (o)->{return o!="asdf";};
        System.out.println(predicate.test("qwer"));
    }
}
```
消费者接口：Consumer<T>
```java
/*
* 消费型接口，只有输入，没有返回值
* */
public class test3 {
    public static void main(String[] args) {
        /*Consumer<String> stringConsumer = new Consumer<String>(){

            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };*/
        Consumer<String> stringConsumer = o->{System.out.println(o);};
        stringConsumer.accept("sdf");
    }
}
```
供给型接口：Supplier<T>
```java
/*
* 供给型接口：没有参数只有返回值
* */
public class test4 {
    public static void main(String[] args) {
        /*Supplier<String> stringSupplier = new Supplier<String>(){
            @Override
            public String get() {
                return "asdf";
            }
        };*/
        Supplier<String> stringSupplier = ()->{return "asdf";};
        System.out.println(stringSupplier.get());
    }
}
```
# Stream流式计算
> 什么式Stream流式计算
大数据： 存储+计算  
集合，MySQL本质就是存储东西的；
计算应该交给流来操作
```java
public class test {
    public static void main(String[] args) {
        User u1 = new User(1,"a",20);
        User u2 = new User(2,"b",22);
        User u3 = new User(3,"c",25);
        User u4 = new User(4,"d",29);
        User u5 = new User(5,"e",18);
        User u6 = new User(6,"f",24);

        //集合用来存储
        List<User> list = Arrays.asList(u1,u2,u3,u4,u5,u6);

        //计算交给流
        list.stream()
                .filter(u->{return u.getId()%2==0;})
                .filter(u->{return u.getAge()>23;})
                .map(u->{return u.getName().toUpperCase();})
                .sorted((uu1,uu2)->{return uu2.compareTo(uu1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```
# ForkJoin
> 什么是ForkJoin

并行执行任务！提高效率，大数据量

> ForkJoin工作特点：工作窃取

如何使用ForkJoin
- forkjoinPool通过它来执行
- 计算任务forkjoinPool.execute(ForkJoinTask task)
- 计算类要继承ForkJoinTask
**ForkJoinDemo**
```java
public class ForkJoinDemo extends RecursiveTask<Long> {
    private Long start;
    private Long end;


    private Long temp=10000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }

    //计算方法
    @Override
    protected Long compute() {
        if((end-start)<temp){
            Long sum=0L;
            for (Long i=start;i<=end;i++){
                sum+=i;
            }
            return sum;
        }else {
            long middle = (start + end) / 2;
            ForkJoinDemo forkJoinDemo = new ForkJoinDemo(start, middle);
            forkJoinDemo.fork();
            ForkJoinDemo forkJoinDemo1 = new ForkJoinDemo(middle, end);
            forkJoinDemo1.fork();
            return forkJoinDemo.join()+forkJoinDemo1.join();
        }
    }
}
```
**test**
```java
public class test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        test3();
    }

    //普通方式
    public static void test1(){
        Long sum = 0L;
        long start = System.currentTimeMillis();
        for (Long i=1L;i<=10_0000_0000;i++){
            sum+=i;
        }
        long end = System.currentTimeMillis();
        System.out.println("执行时间："+(end-start));   //执行时间：12505
    }

    //forkjoin
    public static void test2() throws ExecutionException, InterruptedException {
        Long sum = 0L;
        long start = System.currentTimeMillis();
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> forkJoinDemo = new ForkJoinDemo(1L, 10_0000_0000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(forkJoinDemo);
        submit.get();
        long end = System.currentTimeMillis();
        System.out.println("执行时间："+(end-start));   //执行时间：6018
    }

    //stream 并行流
    public static void test3(){
        Long sum = 0L;
        long start = System.currentTimeMillis();
        LongStream.rangeClosed(0L,10_0000_0000L).parallel().reduce(0,Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("执行时间："+(end-start));         //执行时间：486
    }
}
```
# 异步回调
>Future 设计初衷：对将来的某个事件的结果进行建模
**没有返回值的异步回调**
```java
CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(()->{
    try {
        TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(Thread.currentThread().getName()+"runAsync=>Void");
});
System.out.println("1111111111111");
completableFuture.get();
```
输出结果
```text
1111111111111
ForkJoinPool.commonPool-worker-3runAsync=>Void
```
**带有返回值的异步回调**
```java
CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(()->{
    System.out.println(Thread.currentThread().getName()+"supplyAsync=>Integer");
    //int i = 10/0;
    return 5;
});

System.out.println(completableFuture.whenComplete((t, u) -> {
     System.out.println("t=>" + t);
    System.out.println("u=>" + u);
}).exceptionally((e) -> {
    e.getMessage();
    return 400;
}).get());
```
当无错误时结果
```text
ForkJoinPool.commonPool-worker-3supplyAsync=>Integer
t=>5
u=>null
5
```
当有错误时结果
```java
ForkJoinPool.commonPool-worker-3supplyAsync=>Integer
t=>null
u=>java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
400
```
# JMM
>volatile

volatile是Java虚拟机提供的**轻量级的同步机制**
1.保证可见性
2.不保证原子性
3.禁止指令重排
>什么是JMM

Java内存模型，不存在的东西，概念！约定！
**关于JMM的一些同步的约定**
1.线程解锁前，必须将共享变量立刻刷回主存
2.线程加锁前，必须读取主存中的最新值到工作内存中！
3.加锁和解锁是同一把锁

**8种操作：**
- lock（锁定）：作用于主内存的变量，把一个变量标识为一条线程独占状态。
- unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
- assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
- write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

**Java内存模型还规定了在执行上述八种基本操作时，必须满足如下规则：**

- 如果要把一个变量从主内存中复制到工作内存，就需要按顺寻地执行read和load操作， 如果把变量从工作内存中同步回主内存中，就要按顺序地执行store和write操作。但Java内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行。
- 不允许read和load、store和write操作之一单独出现
- 不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。
- 不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。
- 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
- 一个变量在同一时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。lock和unlock必须成对出现
- 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值
- 如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。
- 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）。

问题：当有两个线程获取主存里的一个资源时，双方不可见，不能及时知道数据已经刷新
# Volatile
>1.保证可见性
```java
public class demo01 {
    //不加volatile 程序会死循环
    //加volatile 可以保证可见性
    private volatile static int num = 0;
    public static void main(String[] args) throws InterruptedException {


        new Thread(()->{
            while (num==0){

            }
        }).start();

        TimeUnit.SECONDS.sleep(1);

        num = 1;
        System.out.println(num);
    }
}
```
>2.不保证原子性(使用原子类来解决)

原子性：不可分割  
线程A在执行任务的时候不能被打扰，不能被分割，要么同时成功，要么同时失败
```java
public class demo02 {
    //不保证原子性
    private volatile static int num = 0;

    public static void add(){
        num++;
    }

    public static void main(String[] args) {
        for(int i=1;i<=20;i++){
            new Thread(()->{
                for (int j=0;j<1000;j++){
                    add();;
                }
            }).start();
        }

        while (Thread.activeCount()>2){
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName()+" "+num);
    }
}



//main 18918
//使用volatile并没有得到预期的输出20000，说明线程之间还是存在问题
```
![](https://note.youdao.com/yws/api/personal/file/WEBb64a65193b1136b7d4a5aa68af8079d6?method=download&shareKey=312c2058f6684875a6bdd2e181070cdc)
**可以看到操作并不是原子性的。需要使用原子类来解决原子性问题**
```java
public class demo02 {
    //不保证原子性
    private volatile static AtomicInteger num = new AtomicInteger();
    public static void add(){
        num.getAndIncrement();    //加1方法，CAS
    }
    public static void main(String[] args) {
        for(int i=1;i<=20;i++){
            new Thread(()->{
                for (int j=0;j<1000;j++){
                    add();
                }
            }).start();
        }
        while (Thread.activeCount()>2){
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+" "+num);
    }
}


//main 20000
//可以看到使用原子类时操作成功，得到预期结果。
```
**这些原子类的底层都直接和操作系统挂钩。在内存中修改值Unsafe类是一个很特殊的存在。**
>指令重排
什么是指令重排？ 

**你写的程序，计算机并不是按照你写的去执行的**  
源代码--->编译器优化重排--->指令并行也可能重排---->内存系统也可能重排---->执行
```java
  int a=2;    //1
  int b=1;    //2
  a=a+5;      //3
  b=a*a;     //4

    //期望的是1234，  但是可能是1324，2134
```
volatile可以避免指令重排：
内存屏障，CPU指令。作用：

1.保证特定的操作的执行顺序！  
2.可以保证某些变量的内存可见性。

# 单例模式
饿汉式
```java
//饿汉式
public class hungry {
    //构造方法私有
    private hungry(){

    }
    //直接创建
    private final static hungry HUNGRY = new hungry();
    //返回
    public static hungry getInstance(){
        return HUNGRY;
    }
}
```
懒汉式
```java
//懒汉式
public class LazyMan {

    private LazyMan(){

    }
    //加volatile保证不发生指令重排
    private volatile static LazyMan lazyMan;
    //在需要的时候在初始化
    public static LazyMan getInstance(){
        if(lazyMan==null){
            synchronized (LazyMan.class){
                if (lazyMan==null){
                    lazyMan =new LazyMan();

                    /*1.分配内存空间
                      2.执行构造方法
                      3.将对象指向这个空间
                      但是在cpu中这3步无法保证原子性，可能发生指令重排，比如132，当执行完3后如果线程b执行，那么会判断不为空，返回lazyMan
                      ,但是此时没有执行构造，会报空指针异常。因此要加volatile确保不进行指令重排
                     */
                }
            }
        }
        return lazyMan;
    }
}
```
# 深入理解CAS
> 什么是CAS

比较并交换，比较工作内存中的值和主存中的值，如果是期望的，则执行操作，否则一直循环。  
**缺点：**  
1.循环耗时。  
2.一次性只能保证一个共享变量  
3.存在ABA问题。
# 原子引用
>解决ABA问题，引入原子引用
```java
public class demo01 {
    public static void main(String[] args) {
        //设置预期值和版本号
        AtomicStampedReference<Integer> atomicStampedReference= new AtomicStampedReference(1,1);

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();
            System.out.println("a1=>"+stamp);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //进行操作时变更期望值，和版本号
            atomicStampedReference.compareAndSet(1,2, atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
            System.out.println("a2=>"+atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(2,1,atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
            System.out.println("a3=>"+atomicStampedReference.getStamp());
        },"a").start();


        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();
            System.out.println("b1=>"+stamp);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //根据版本号来确认是否有过ABA问题
            atomicStampedReference.compareAndSet(1,6,stamp,stamp+1);
            System.out.println("b2=>"+atomicStampedReference.getStamp());

        },"b").start();
    }
}
```
# 各种锁的理解
## 公平锁，非公平锁
公平锁：非常公平，不能插队，必须先来后到
非公平锁：非常不公平，可以插队(默认)
```java
public ReentrantLock() {   //默认非公平锁
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {    //设置为true则为公平锁
    sync = fair ? new FairSync() : new NonfairSync();
}
```
## 可重入锁

在拿到锁时也会将里面的锁拿到
>synchronized
```java
public class demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sms();
        },"A").start();


        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}
class Phone{
    public synchronized void sms(){
        System.out.println(Thread.currentThread().getName()+"sms");
        call();     //也有锁
    }
    public synchronized void call(){
        System.out.println(Thread.currentThread().getName()+"call");
    }
}
```
>lock
```java
public class demo02 {
    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(()->{
            phone.sms();
        },"A").start();


        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}
class Phone2{
    Lock lock = new ReentrantLock();   //锁必须配对，否则会死锁
    public void sms(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"sms");
            call();     //也有锁
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public void call(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"call");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```
## 自旋锁
```java
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```
通过自旋锁来实现一个锁
```java
/*
* 自旋锁
* */
public class spinlock {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();
    //加锁
    public void mylock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"==>mylock");
        while (!atomicReference.compareAndSet(null,thread)){
        }
    }
    //解锁
    public void myunlock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"==>myunlock");
        atomicReference.compareAndSet(thread,null);
    }
}
```
测试
```java
public class testSpinlock {
    public static void main(String[] args) throws InterruptedException {
        spinlock lock = new spinlock();
        new Thread(()->{
            lock.mylock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myunlock();
            }
        },"A").start();

        TimeUnit.SECONDS.sleep(1);
        new Thread(()->{
            lock.mylock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myunlock();
            }
        },"B").start();
    }
}


//结果
A==>mylock   //A拿到锁之后上锁，执行TimeUnit.SECONDS.sleep(3);
B==>mylock   //B在1秒后拿到锁，但是已经被A锁上，自旋等待
A==>myunlock //A休眠3秒后解锁,将锁释放
B==>myunlock  // B退出自旋，进行休眠3秒后，也释放锁
```
# 死锁
```java
public class diedlock01 {
    public static void main(String[] args) {
        String A = "lockA";
        String B = "lockB";
        new Thread(new MyThread(A,B)).start();    //第一个线程获取A资源进行加锁，然后尝试获取B资源
        new Thread(new MyThread(B,A)).start();    //第二个线程获取B资源进行加锁，然后尝试获取A资源
        //结果进入死锁状态
    }
}
class MyThread implements Runnable{
    private String lockA;
    private String lockB;
    public MyThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }
    @Override
    public void run() {
        synchronized (lockA){
            System.out.println(Thread.currentThread().getName()+"lock:"+lockA+"get"+lockB);
            try {
                TimeUnit.SECONDS.sleep(2);      //取得A锁之后休眠2秒，再去获取B锁
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lockB){

            }
        }
    }
}
```
>解决问题

1.使用 `jps -l` 定位进程号
```text
PS D:\2021\redis\redisstudy> jps -l
3024 diedlock.diedlock01
4260 org.jetbrains.jps.cmdline.Launcher
7112 jdk.jcmd/sun.tools.jps.Jps
7272 org.jetbrains.idea.maven.server.RemoteMavenServer36
9384
```
2.使用 `jstack 进程号`查看进程信息
```text
Java stack information for the threads listed above:
===================================================
"Thread-0":
        at diedlock.MyThread.run(diedlock01.java:40)
        - waiting to lock <0x00000006d53b4608> (a java.lang.String)      //等待的锁
        - locked <0x00000006d53b45d8> (a java.lang.String)     //持有的锁
        at java.lang.Thread.run(java.base@11.0.1/Thread.java:834)
"Thread-1":
        at diedlock.MyThread.run(diedlock01.java:40)
        - waiting to lock <0x00000006d53b45d8> (a java.lang.String)     //等待的锁
        - locked <0x00000006d53b4608> (a java.lang.String)     //持有的锁
        at java.lang.Thread.run(java.base@11.0.1/Thread.java:834)

Found 1 deadlock.      //发现一个死锁
```