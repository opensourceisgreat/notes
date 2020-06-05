[toc]



# 1、继承Thread类

start()开启一个线程，交给cpu去调，具体执行看cpu什么时候有时间去执行就执行，==不保证立即执行==

使用run()方法就是按普通的程序顺序执行，不会开启线程

![1.PNG](https://note.youdao.com/yws/api/personal/file/6222BF779D73470AAD7C480CB76144C0?method=download&shareKey=0a83fd3a028700032d60d816ba9f7143)

![2.PNG](https://note.youdao.com/yws/api/personal/file/C13A79740BB84B6287931A2A1CBFAA72?method=download&shareKey=d456ab320be2c87705b9f14b27117768)

==死亡之后无法在切回新生状态==

![3.png](https://note.youdao.com/yws/api/personal/file/9C7975B056A54F788CF1D8DAF522A4E3?method=download&shareKey=00e961388ce8907aa853e8bccb8648eb)



## 1.1终止线程

```java
/**
 * 终止线程
 * 1、正常执行完
 * 2、外部干涉
 * 3、禁止使用stop(),destroy()
 * @author Date 2019/7/3-10:52
 */
public class TerminateThread implements Runnable{
    private boolean flag=true;
    private int i=0;
    @Override
    public void run() {
        while (flag){

            System.out.println("继续执行"+i++);
        }
    }

    public void terminate(){
        flag=false;
    }

    public static void main(String[] args) {
        TerminateThread terminateThread=new TerminateThread();
        new  Thread(terminateThread).start();
        for (int i=0;i<=1000000;i++){
            if (i == 1000000) {
                terminateThread.terminate();
                System.out.println("停止");
            }
        }
    }
}
```

## 1.2sleep()阻塞，静态方法

运行--->阻塞

阻塞结束，线程会到就绪状态，`Thread.sleep()`写在线程体中（run中）

## 1.3yield

运行-->就绪

静态方法，`Thread.yield()`写在线程体中

## 1.4join()

合并线程，此线程执行完成后再执行其他线程，其他线程阻塞

实例方法

```java
Thread t=new Thread(()->{});
t.start();

t.join();//插队，main线程就被阻塞了
```

## 1.5优先级

```java
//NORM_PRIORITY 5 默认
//MIN_PRIORITY 1
//MAX_PRIORITY 10
//只是概率，不代表绝对的优先执行顺序
Thread.currentThread().getPriority();
//设置优先级在启动之前
t.setPriority(Thread.MAX_PRIORITY)
t.start();
```

## 1.6守护线程

![4.png](https://note.youdao.com/yws/api/personal/file/845DF17F1B504224B68E53D0F4351F6F?method=download&shareKey=96b1be8ce3ac9f21d35663dfbc82739a)

为用户线程服务的线程

默认我们的线程都是用户线程

```java
t.setDaemon(true);//将用户线程调整为守护
t.start();
```

## 1.7其他方法

默认线程名字是Thread-0,Thread-1.....等

`t.setName()`设置代理名称

![5.png](https://note.youdao.com/yws/api/personal/file/3558461AD511407DAB2C466950B37ADB?method=download&shareKey=9a0197bb64a2416afdac8fb265111638)



# 2、实现Runnable接口(多使用)

# 3、线程同步

并发：一个对象被多个线程同时操作

形成对列，给对象加锁就能完成线程同步，每个对象都自带排它锁

线程不安全原因之一：线程会将公共资源备份到自己的工作区，待处理完在将资源修改结果反馈给公共资源区，所以最终多个线程操作资源时会产生错误的结果

![6.PNG](https://note.youdao.com/yws/api/personal/file/6B87DF8090DC4727B122A38992D179FC?method=download&shareKey=73aef29013c1a18e765f5372ebd9b671)

锁机制虽然耗费资源但是可以忍

`synchronized`关键字，==锁的是谁？要清楚==

一个对象中可能有多个属性，有的属性只需要读，有的属性可能要修改，锁的时候全锁就影响效率了

## 3.1、同步方法

`synchronized`加在方法返回值前

线程执行该方法时会判断方法和对象有没有关联(即有没有锁)，是否锁了对象的资源，此时的对象就是this，而方法里面都是对象相关的资源，如果不是就锁失败了。记住==锁的是对象==,操作这个方法时看的是对象能不能用

## 3.2、同步块

![7.PNG](https://note.youdao.com/yws/api/personal/file/6EBA501F0D354C9EBC888E7E9D7442D5?method=download&shareKey=77783f079aa1f6ae6297fced0fc1a3bb)



抢火车票案例：

```java
package com.whpu;

/**
 * @author Date 2019/7/17-13:57
 */
public class SynBlock {
    public static void main(String[] args) {
        WebTicket webTicket = new WebTicket(10);
        new Thread(webTicket, "x").start();
        new Thread(webTicket, "y").start();
    }
}

class WebTicket implements Runnable {
    public WebTicket(int ticketNums) {
        this.ticketNums = ticketNums;
    }

    private Integer ticketNums;
    private boolean flag = true;

    @Override
    public void run() {
        while (flag) {
            test4();
        }
    }


    //线程安全 同步
    public synchronized void test1() {
        if (this.ticketNums <= 0) {
            flag = false;
            return;
        }
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "---" + ticketNums--);
    }

    //使用同步块 线程安全同步，范围太大，效率低
    public void test2() {
        synchronized (this) {
            if (this.ticketNums <= 0) {
                flag = false;
                return;
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "---" + ticketNums--);
        }
    }

    //使用同步块 同步块锁的必须是对象，所以要转换，这里锁了ticketNums，线程仍然不安全,被锁的对象是不变的，只能变属性，1,2,3是不同的对象了
    //而且flag没有锁
    public void test3() {
        synchronized (ticketNums) {
            if (this.ticketNums <= 0) {
                flag = false;
                return;
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "---" + ticketNums--);
        }

    }

    //线程不安全，没有锁完整
    public void test4() {
        synchronized (this) {
            if (this.ticketNums <= 0) {
                flag = false;
                return;
            }
        }
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "---" + ticketNums--);
    }
    //线程安全尽可能锁定合理的范围,这里的范围指的是数据的完整性不是代码量的多少
    //double checking
    //这种情况要多注意，多使用同步块
    public void test5() {
        //考虑没有票的情况
        if (this.ticketNums <= 0) {
            flag = false;
            return;
        }
        synchronized (this) {
            if (this.ticketNums <= 0) {//考虑最后一张票
                flag = false;
                return;
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "---" + ticketNums--);
        }
    }
}

```

订购电影票：

```java
package com.whpu;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * @author Date 2019/7/17-15:32
 */
public class HappyCinema {
    public static void main(String[] args) {
        //可用的位置
        List<Integer> available=Arrays.asList(1,2,3,6,7);
        //顾客需要的位置
        List<Integer> seats1=Arrays.asList(3);
        List<Integer> seats2=Arrays.asList(1,2,6);
        Cinema c = new Cinema(available, "happy");
        new Thread(new Customer(c, seats1), "ming").start();
        new Thread(new Customer(c, seats2), "hong").start();

    }

}

//顾客
class Customer implements Runnable {
    Cinema cinema;
    List<Integer> seats;

    public Customer(Cinema cinema, List<Integer> seats) {
        this.cinema = cinema;
        this.seats = seats;
    }

    @Override
    public void run() {
        //注意使用同步块锁定cinema，无脑使用同步方法会让线程锁定顾客对象，我们要锁的是cinema才对
        synchronized (cinema) {
            boolean flag = cinema.bookTickets(seats);
            if (flag) {
                System.out.println("出票成功" + Thread.currentThread().getName() + "->位置：" + seats);

            } else {
                System.out.println("出票失败" + Thread.currentThread().getName() + "->位置不够");

            }
        }
    }
}

//影院
class Cinema {
    List<Integer> available;//可用的位置
    String name;//名称

    public Cinema(List<Integer> available, String name) {
        this.available = available;
        this.name = name;
    }

    //购票
    public boolean bookTickets(List<Integer> seats) {
        System.out.println("可用位置：" + available);
        List<Integer> copy = new ArrayList<Integer>();
        copy.addAll(available);

        //相减
        copy.removeAll(seats);

        //判断大小
        if (available.size() - copy.size() != seats.size()) {
            return false;

        }
        //成功
        available=copy;
        return true;

    }
}

```

购票代码：可以开阔视野，和上面的代码进行比较

```java
package com.whpu;


/**
 * @author Date 2019/7/18-10:33
 */
public class Happy12306 {
    public static void main(String[] args) {
    Web web=new Web(4,"happy");
    //同一个角色(web)，多分代理(Passenger)
    new Passenger(web,"hong",3).start();
    new Passenger(web,"ming",1).start();
    }
}

//顾客
class Passenger extends Thread {
    int seats;

    public Passenger(Runnable target, String name, int seats) {
        super(target, name);
        this.seats = seats;
    }

}

//火车票网
class Web implements Runnable {
    int available;
    String name;

    public Web(int available, String name) {
        this.available = available;
        this.name = name;
    }

    @Override
    public void run() {
        Passenger p= (Passenger) Thread.currentThread();
        boolean flag = this.bookTickets(p.seats);
        if (flag) {
            System.out.println("出票成功" + Thread.currentThread().getName() + "->位置：" + p.seats);

        } else {
            System.out.println("出票失败" + Thread.currentThread().getName() + "->位置不够");

        }
    }

    //购票
    public synchronized boolean bookTickets(int seats) {
        System.out.println("可用位置：" + available);
        if (seats > available) {
            return false;
        }
        available -= seats;
        return true;

    }
}
```





## 3.3、并发容器

`CopyOnWriteArrayList`，并发编程操作容器时可以使用这个容器

# 4、死锁

过多的同步可能造成相互不是放资源，从而相互等待，一般发生在同步中持有多个对象的锁

避免：不要在同一个代码块中，同时持有多个对象的锁

```java
package com.whpu;

/**
 * @author Date 2019/7/18-14:02
 */
public class DeadLock {
    public static void main(String[] args) {
        Markup g1=new Markup(1,"hong");
        Markup g2=new Markup(0,"ming");
        g1.start();
        g2.start();
    }
}

//口红
class Lipstick {

}

//镜子
class Mirror {

}

class Markup extends Thread {
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();
    //选择
    int choice;
    String girl;

    public Markup(int choice, String girl) {
        this.choice = choice;
        this.girl = girl;
    }

    @Override
    public void run() {
        //化妆
        markup();
    }

    private void markup() {
        if (choice == 0) {
            synchronized (lipstick) {
                System.out.println(this.girl + "获得口红");
                //2秒后想要拥有镜子
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (mirror){
                    System.out.println(this.girl + "镜子");
                }
            }
        }else {
            synchronized (mirror) {
                System.out.println(this.girl + "镜子");
                //1秒后想要拥有镜子
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
//                synchronized (lipstick){
//                    System.out.println(this.girl + "获得口红");
//                }
            }
            synchronized (lipstick){
                System.out.println(this.girl + "获得口红");
                }
        }
    }
}


```

# 5、线程协作

![8.PNG](https://note.youdao.com/yws/api/personal/file/3F87C3EF2E964EF28A360A06E6AC0E0D?method=download&shareKey=2b07b759cc709cfc8c34c65e79447de6)

![9.PNG](https://note.youdao.com/yws/api/personal/file/6D5801F1F2B2442A8A05A4EAF4CB66C2?method=download&shareKey=739ac60488ec8b947ac5fc9ba659be49)

解决方式2：并发协作模型“生产者/消费者模式”------信号灯法

![10.PNG](https://note.youdao.com/yws/api/personal/file/096F85EA26064828BCC837612BBBDEB3?method=download&shareKey=d75616f4f264777b20644b525f1e1252)

案例：

```java
package com.whpu;

/**
 * @author Date 2019/7/18-15:49
 * 管程法：借助缓冲区
 */
public class CoTest01 {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        new Productor(container).start();
        new Consumer(container).start();
    }
}

//生产者
class Productor extends Thread {
    SynContainer container;

    public Productor(SynContainer container) {
        this.container = container;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("生产-----" + i + "个馒头");
            container.push(new Steamedbun(i));
        }
    }
}

//消费者
class Consumer extends Thread {
    SynContainer container;

    public Consumer(SynContainer container) {
        this.container = container;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费-----" + container.pop().id + "个馒头");
        }
    }
}

//缓冲区
class SynContainer {
    Steamedbun[] buns = new Steamedbun[10];
    int count = 0;


    //生产
    public synchronized void push(Steamedbun bun) {
        //容器存在空间可以生产
        if (count == buns.length) {
            try {
                this.wait();//没有空间线程阻塞
            } catch (InterruptedException e) {

            }
        }
        buns[count] = bun;
        count++;
        //存在数据，唤醒消费
        this.notifyAll();
    }

    //获取
    public synchronized Steamedbun pop() {
        //没有数据就等待
        if (count == 0) {

            try {
                this.wait();
            } catch (InterruptedException e) {
            }
        }
        Steamedbun bun = buns[--count];
        this.notifyAll();//存在空间了唤醒生产
        return bun;
    }

}

//馒头
class Steamedbun {
    int id;

    public Steamedbun(int i) {
        this.id = i;
    }
}
```



```java
package com.whpu;

/**
 * @author Date 2019/7/18-16:19
 * 生产者消费者：信号灯
 * 标志位
 */
public class CoTest02 {
    public static void main(String[] args) {
        Tv tv=new Tv();
        new Player(tv).start();
        new Watcher(tv).start();
    }
}

//生产者 演员
class Player extends Thread {
    Tv tv;

    public Player(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i % 2 == 0) {
                this.tv.play("奇葩说");
            } else {
                this.tv.play("太污了");
            }
        }
    }
}

//消费者 观众
class Watcher extends Thread {
    Tv tv;

    public Watcher(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            this.tv.watch();
        }
    }
}

class Tv {
    String voice;
    //信号灯
    //T表示演员表演 观众等待
    //F表示观众观看 演员等待
    boolean flag = true;

    //表演
    public synchronized void play(String voice) {
        //演员等待
        if (!flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {

                e.printStackTrace();
            }
        }
        System.out.println("表演了" + voice);
        this.voice = voice;
        this.notifyAll();
        this.flag=!this.flag;

    }

    //观看
    public synchronized void watch() {
        //观众等待
        if (flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {

                e.printStackTrace();
            }
        }
        System.out.println("听到了" + voice);
        this.notifyAll();
        this.flag=!this.flag;
    }
}

```





![11.PNG](https://note.youdao.com/yws/api/personal/file/28C26566AD8348539D8F5F2A59E7A613?method=download&shareKey=36c7e07ce7c2e09cc1c6bfb9902cfd85)

==wait会释放锁==

# 6、高级主题

## 6.1、任务定时调度

![12.PNG](https://note.youdao.com/yws/api/personal/file/16BAEAC46B264C68A371ED0DA8505886?method=download&shareKey=261b7368d6052d9514ff1b14b9bccc20)

​	Quartz已经应用到Spring框架中了，maven可以导包

![13.PNG](https://note.youdao.com/yws/api/personal/file/B6F344D6396B4753A10D1B9F0C9E3BCA?method=download&shareKey=e6e54ac3bc154c832fabcc543db70644)

写的时候官方文档会有例子，按照文档写

[Quartz官网下载源码，含有demo](http://www.quartz-scheduler.org/downloads/)

## 6.2、HappenBefore 

![14.PNG](https://note.youdao.com/yws/api/personal/file/A80C95A09EA940E0BE0F95EE8EA58B70?method=download&shareKey=c46ff738a7a00a370c1c5d7667e7c065)

![15.PNG](https://note.youdao.com/yws/api/personal/file/13BCC86B009B493AA4329070AF7D5AD1?method=download&shareKey=3402e2291a9fb7bc2c5d0b2a0252b3ea)

遇到的不是很多

## 6.3、volatile(不多见了)

![16.PNG](https://note.youdao.com/yws/api/personal/file/2C4C05E25A884C3AA8E3DA2FF55523C4?method=download&shareKey=cac26b0dac84c6278c97b523527db336)

```java
package com.whpu;

/**
 * volatile保证数据的同步性，也就是可见性
 * 比起synchronized，小巫见大巫
 * @author Date 2019/7/19-11:26
 * 不加volatile，cpu忙的不可开交，没时间同步num,一直循环
 */
public class VolatileTest {
    private volatile static int num=0;
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (num==0){

            }
        }).start();
        Thread.sleep(1000);
        num=1;
    } 
}
```

## 6.4、单例模式（实现方法很多，这里用其中一种方法实现）

```java
package com.whpu;

/**
 * (DCL)单例模式：在多线程环境下对外存在一个对象
 * 1、构造器私有化--避免外部new构造器
 * 2、提供私有的静态属性--存储对象的地址
 * 3、提供公共的静态方法--获取属性
 * @author Date 2019/7/19-11:42
 */
public class DoubleCheckedLocking {
    //2、提供私有的静态属性
    //没有volatile可能其他线程访问一个没有初始化的对象
    //因为系统进行指令重排可能会先先进行instance赋值
    private static volatile DoubleCheckedLocking instance;

    //1、构造器私有化
    private DoubleCheckedLocking(){

    }
    //3、提供公共的静态方法
    public static DoubleCheckedLocking getInstance(){
        if (null != instance) {//避免不必要的同步，已经存在对象
            return instance;
        }
        //静态方法同步块，使用类模子
        synchronized (DoubleCheckedLocking.class){
            if (null == instance) {
                instance=new DoubleCheckedLocking();
            }

        }
        return instance;
    }

    public static void main(String[] args) {
        Thread t=new Thread(()->{
            System.out.println(DoubleCheckedLocking.getInstance());
        });
        t.start();
        System.out.println(DoubleCheckedLocking.getInstance());
    }
}

```

## 6.5、ThreadLocal

![17.PNG](https://note.youdao.com/yws/api/personal/file/3A380AAFBBE14A719215BA3C6254EA8D?method=download&shareKey=fb85ef9476c33488c1ad363617457a81)

```java
package com.whpu;

/**
 * @author Date 2019/7/19-14:20
 * ThreadLocal:每个线程自身的存储本地、局部区域
 * get/set/initialValue
 */
public class ThreadLocalTest01 {
    //    private static ThreadLocal<Integer> threadLocal=new ThreadLocal<>();
    //更改初始化值
//    private static ThreadLocal<Integer> threadLocal = new ThreadLocal() {
//
//        @Override
//        protected Integer initialValue() {
//            return 200;
//        }
//    };
    private static ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 200);

    public static void main(String[] args) {

        System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());
        threadLocal.set(99);
        System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());
        new Thread(new MyRun()).start();
        new Thread(new MyRun()).start();
        new Thread(new MyRun()).start();
        new Thread(new MyRun()).start();
        new Thread(new MyRun()).start();
    }

    public static class MyRun implements Runnable {

        @Override
        public void run() {
            threadLocal.set((int) (Math.random() * 99));
            System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());
        }
    }

}

```

```java
package com.whpu;

/**
 * @author Date 2019/7/19-14:20
 * ThreadLocal:每个线程自身的上下文
 * get/set/initialValue
 */
public class ThreadLocalTest02 {

    private static ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 1);

    public static void main(String[] args) {
        //new MyRun()还是在main中调用属于main，构造器中的赋值时针对main
        //run()方法属于MyRun线程上下文
        //谁调用属于谁
        new Thread(new MyRun()).start();
        new Thread(new MyRun()).start();
    }

    public static class MyRun implements Runnable {
        public MyRun() {
            threadLocal.set(-100);
            System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());

        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());
        }
    }

}

```

```java
package com.whpu;

/**
 * @author Date 2019/7/19-14:20
 * InheritableThreadLocal:继承上下文环境的数据，拷贝一份给子线程（是拷贝不是共享）
 */
public class ThreadLocalTest03 {

    private static ThreadLocal<Integer> threadLocal = new InheritableThreadLocal<>();

    public static void main(String[] args) {

        threadLocal.set(2);
        System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());
        //此线程有main线程开辟
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());
            threadLocal.set(200);
            System.out.println(Thread.currentThread().getName() + "-->" + threadLocal.get());
        }).start();

    }
}

```
# 7、可重入锁

![18.PNG](https://note.youdao.com/yws/api/personal/file/50D658A2C8334A65B98861ECD0A141F8?method=download&shareKey=501b8433a399687922d3ed17c2977394)

手动打造锁

```
package com.whpu;

/**
 * @author liang
 * Date 2019/7/21-10:12
 * 不可重入锁：锁不可以延续使用
 */
public class LockTest {
    Lock lock=new Lock();
    public void a() throws InterruptedException {
        lock.lock();
        doSomething();
        lock.unlock();
    }
    //不可重入
    public void doSomething() throws InterruptedException {
        lock.lock();
        //.......
        lock.unlock();
    }
    public static void main(String[] args) throws InterruptedException {
        //死循环，不可重入锁，进入a(),但是不能doSomething;
        LockTest lockTest = new LockTest();
        lockTest.a();
        lockTest.doSomething();
    }
}
//不可重入锁
class Lock {
    //是否占用
    private boolean isLocked = false;
    //使用锁
    public synchronized void lock() throws InterruptedException {
        while (isLocked){
            wait();
        }
        isLocked=true;
    }
    //释放锁
    public synchronized void unlock(){
        isLocked=false;
        notify();
    }
}

```
```
package com.whpu;

/**
 * @author liang
 * Date 2019/7/21-10:12
 * 可重入锁：锁可以延续使+计数器
 */
public class LockTest2 {
    ReLock lock = new ReLock();

    public void a() throws InterruptedException {
        lock.lock();
        System.out.println(lock.getHoldCount());
        doSomething();
        lock.unlock();
        System.out.println(lock.getHoldCount());
    }

    //不可重入
    public void doSomething() throws InterruptedException {
        lock.lock();
        System.out.println(lock.getHoldCount());
        //.......
        lock.unlock();
        System.out.println(lock.getHoldCount());
    }

    public static void main(String[] args) throws InterruptedException {

        LockTest2 lockTest = new LockTest2();
        lockTest.a();

        Thread.sleep(1000);
        System.out.println(lockTest.lock.getHoldCount());
    }
}

//可重入锁
class ReLock {
    //是否占用
    private boolean isLocked = false;
    //存储线程
    private Thread lockedBy = null;
    private int holdCount = 0;

    //使用锁
    public synchronized void lock() throws InterruptedException {
        Thread t = Thread.currentThread();
        while (isLocked && lockedBy != t) {
            wait();
        }
        isLocked = true;
        lockedBy = t;
        holdCount++;
    }

    //释放锁
    public synchronized void unlock() {
        if (Thread.currentThread() == lockedBy) {
            holdCount--;
            if (holdCount == 0) {
                isLocked = false;
                notify();
                lockedBy = null;
            }
        }

    }

    public int getHoldCount() {
        return holdCount;
    }
}

```
java提供了可重入锁ReentrantLock，上面是手写

```
package com.whpu;


import java.util.concurrent.locks.ReentrantLock;

/**
 * @author liang
 * Date 2019/7/21-10:12
 * 可重入锁：锁可以延续使+计数器
 */
public class LockTest2 {
    ReentrantLock lock=new ReentrantLock();

    public void a() throws InterruptedException {
        lock.lock();
        System.out.println(lock.getHoldCount());
        doSomething();
        lock.unlock();
        System.out.println(lock.getHoldCount());
    }

    //不可重入
    public void doSomething() throws InterruptedException {
        lock.lock();
        System.out.println(lock.getHoldCount());
        //.......
        lock.unlock();
        System.out.println(lock.getHoldCount());
    }

    public static void main(String[] args) throws InterruptedException {

        LockTest2 lockTest = new LockTest2();
        lockTest.a();

        Thread.sleep(1000);
        System.out.println(lockTest.lock.getHoldCount());
    }
}

```

# 8、CAS

![19.PNG](https://note.youdao.com/yws/api/personal/file/BF8F384754E0418CB04EDB372998913C?method=download&shareKey=fbb2fff11fb8167eb18115f946b26762)

```
package com.whpu;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author liang
 * Date 2019/7/21-14:16
 * CAS：比较并交换
 */
public class CAS {
    //库存
    private static AtomicInteger stock = new AtomicInteger(5);

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                //模拟延时
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Integer left = stock.decrementAndGet();
                if (left < 1) {
                    System.out.println("抢完了");
                    return;
                }
                System.out.print(Thread.currentThread().getName()+"抢到了");
                System.out.println("---还剩"+left);
            }).start();
        }
    }
}

```

# 9、JUC（未来高级工程师要掌握的内容）
并发编程中很常用的工具类