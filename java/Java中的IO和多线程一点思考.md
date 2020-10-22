### 参考文档

[BIO,NIO,AIO知识](https://snailclimb.gitee.io/javaguide/#/docs/java/BIO-NIO-AIO?id=_1-bio-blocking-io)

[各种IO模型讲解](https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247484746&idx=1&sn=c0a7f9129d780786cabfcac0a8aa6bb7&source=41#wechat_redirect)

[多线程的实践](https://snailclimb.gitee.io/javaguide/#/./docs/java/Multithread/best-practice-of-threadpool)

### 知识补充

1. 在 Linux 这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的系统函数。
2. 使用线程池需要命名，便于分析

### 多线程静态代理

```java
public class StaticProxy {
    public static void main(String[] args) {
        
        //创建代理对象时，引用不要用接口，因为代理类通常会除了有公共接口方法外还有其他方法，用接口就没法调用其他方法了，
        //创建线程就用的start()方法，而不是仅仅通过代理对象Thread调用run()
        new MerryCompany(new You()).marry();
        new Thread(()->{}).start();
        
        //对比发现Thread就是按静态代理设计的
        //实现静态代理模式时有三个要素
		//1.真实角色(委托对象)
		//2.代理角色
		//3.共同实现的接口
		//对比以实现Runnable接口的形式创建多线程,可以发现,代理角色Thread类不需要我们创建,我们只需要写委托对象
    	//实现Runnable接口.把委托对象的引用传递给Thread,借助Thread对象来开启线程即可
    }
}
//公共接口
interface Marry{
    public void marry();
}
//真实角色
class You implements Marry{

    @Override
    public void marry() {
        System.out.println("结婚");
    }
}
//代理角色
class MerryCompany implements Marry{
    private Marry target;

    public MerryCompany(Marry target) {
        this.target = target;
    }

    @Override
    public void marry() {
        System.out.println("before marry");
        target.marry();
        System.out.println("after marry");
    }
}
```

### 多线程wait()

> 线程进入wait状态后释放锁，等待被唤醒后，会进入阻塞状态（就是要去抢锁的状态），重新获取到锁后，继续执行wait后的代码

### ABA问题

一个CAS操作的过程可以用以下c代码表示: [[1\]](https://zh.wikipedia.org/wiki/比较并交换)

```c
1 int cas(long *addr, long old, long new)
2 {
3     /* Executes atomically. */
4     if(*addr != old)
5         return 0;
6     *addr = new;
7     return 1;
8 }
```

ABA问题是无锁结构实现中常见的一种问题，可基本表述为：

1. 进程P1读取了一个数值A
2. P1被挂起(时间片耗尽、中断等)，进程P2开始执行
3. P2修改数值A为数值B，然后又修改回A
4. P1被唤醒，比较后发现数值A没有变化，程序继续执行。

对于P1来说，数值A未发生过改变，但实际上A已经被变化过了，继续使用可能会出现问题。在CAS操作中，由于比较的多是指针，这个问题将会变得更加严重。试想如下情况：

```
   top
    |
    V   
  0x0014
| Node A | --> |  Node X | --> ……
```

有一个栈(先入后出)中有top和节点A，节点A目前位于栈顶top指针指向A。现在有一个进程P1想要pop一个节点，因此按照如下无锁操作进行

```
pop()
{
  do{
    ptr = top;            // ptr = top = NodeA
    next_prt = top->next; // next_ptr = NodeX
  } while(CAS(top, ptr, next_ptr) != true);
  return ptr;   
}
```

而进程P2在执行CAS操作之前打断了P1，并对栈进行了一系列的pop和push操作，使栈变为如下结构：

```
   top
    |
    V  
  0x0014
| Node C | --> | Node B | --> |  Node X | --> ……
```

进程P2首先pop出NodeA，之后又Push了两个NodeB和C，由于内存管理机制中广泛使用的内存重用机制，导致NodeC的地址与之前的NodeA一致。

这时P1又开始继续运行，在执行CAS操作时，由于top依旧指向的是NodeA的地址(实际上已经变为NodeC)，因此将top的值修改为了NodeX，这时栈结构如下：

```
                                   top
                                    |
   0x0014                           V
 | Node C | --> | Node B | --> |  Node X | --> ……
```

经过CAS操作后，top指针错误的指向了NodeX而不是NodeB。

### 可见性

在 Java 中 volatile、synchronized 和 final 实现可见性。

[参考文档](https://www.cnblogs.com/zhengbin/p/5654805.html)