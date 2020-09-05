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