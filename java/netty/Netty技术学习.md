## 参考资源

[https://www.bilibili.com/video/BV1WJ411k7Hk?p=92](https://www.bilibili.com/video/BV1WJ411k7Hk?p=92)

[Linux五种IO模型](https://juejin.im/post/6844903687626686472)

## 一些基础知识

### NIO 不同语境下的说法不同

在 OS 中 NONBLOCKING

在 JAVA 中 NEW IO

JAVA 中使用 NIO 时，可以配置调用系统调用中的非阻塞Socket,也就是说 Java 中 NIO 既可以使用阻塞也可使用非阻塞

当使用非阻塞配置后，JAVA 不会阻塞在 accept 而是返回客户端对象或者 null 

返回的 客户端对象也设置为非阻塞，加入到客户端队列，让后循环遍历客户端调用每个客户端 read 也就不阻塞了，循环读取所有客户端的数据。

伪代码：

```java
while(true){
    //1、accept 获取客户端连接
    //加入客户端到队列
    //2、循环遍历队列，看每个客户端是否有数据输入
}
```

**完成了一个线程，处理多个连接和IO的功能**

NIO 弊端：从上述代码可以发现，每次while循环的内部都会有O(n)的系统调用，并发量大的时候，简直是灾难，因为可能压根儿没有数据输入

### 多路复用器

**select,poll,epoll(linux) 都是同步IO模型（因为最终还是需要程序自己去读数据）**

java 中的 NIO 可以使用 `Selector.open()`来使用多路复用模型,不同OS会使用不同的多路复用器

OS 中的 NIO 其实和多路复用器关系不大，OS 中 NIO 简单想就是 accept 系统调用会立马返回，read 系统调用也是立马返回

JAVA 中的NIO 使用系统调用（select或poll）时，它们会返回当前有数据的socket client 的文件描述符给应用程序，相当于只调用了一次系统调用，不会像之前循环调用系统调用。然后，程序自己再去读取对应socket文件数据

JAVA NIO 中写程序时会将服务端（用于接收连接的socket）和客户端(用于传数据的socket)都注册到多路复用器中，这样每次只需要调用多路复用器，通过返回的文件描述符集合，判断每个文件接收到的数据是建立连接请求还是要读数据，然后做出相应操作（建立连接请求中，我们的操作就是将客户端socket注册到多路复用器中）

**弊端**

每次都要传递数据给多路复用器监听了哪些文件

每次调用都要触发内核遍历



epoll 解决上述弊端，epoll它只会对"活跃"的socket进行操作

- epoll_create 在内核中创建一个记事本，将监听的socket文件描述符放入
- epoll_ctl（可以添加客户端socket到记事本，或者修改,删除记事本中的内容）
- epoll_wait 查询哪个socket 有数据来了，建立连接请求就调用epoll_ctl ,读数据请求就让程序去对应文件读

记事本用红黑树实现，队列就是链表



**epoll机制是Linux下一种高效的IO复用方式，相较于select和poll机制来说。其高效的原因是将基于事件的fd放到内核中来完成，在内核中基于红黑树+链表数据结构来实现，链表存放有事件发生的fd集合，然后在调用epoll_wait时返回给应用程序，由应用程序来处理这些fd事件。**

JDK 对epoll的实现由BUG，参考

[https://www.cnblogs.com/luoxn28/p/11872858.html](https://www.cnblogs.com/luoxn28/p/11872858.html),

[JDK Epoll空轮询bug](https://www.jianshu.com/p/3ec120ca46b2)

[内核如何阻塞和唤醒进程](https://www.jianshu.com/p/11adc779719a),**线程在加入到等待队列的同时向内核注册了一个回调函数，告诉内核我在等待这个 Socket 上的数据，如果数据到了就唤醒我。这样当网卡接收到数据时，产生硬件中断，内核再通过调用回调函数唤醒进程。**

[epoll详解](https://zhuanlan.zhihu.com/p/63179839)

内核中维护一个中断表，把网卡中断这个操作和回调函数绑定注册到表中就能在网卡中断时执行回调函数，完成进程唤醒

参考文档,[网络IO](https://blog.csdn.net/sinat_42483341/category_10210911.html)

### NIO 中 Buffer 的一些问题

**前置知识**

DirectByteBuffer是堆外内存，不是 JVM 中的堆，是指直接内存，通过该对象提供的操作是用来操作堆外的内存，**该对象本身是在堆中**，回收该对象时，调用JNI来回收堆外内存，JVM GC本身是没法回收堆外内存的

参考文档 [Java堆外内存](https://blog.csdn.net/qq_21125183/article/details/88701156)

[Java堆外内存的回收机制](https://www.jianshu.com/p/e45c1f8f3ada)

[深入浅出MappedByteBuffer](https://www.jianshu.com/p/f90866dcbffc)

![image-20200929190722907](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200929190730.png)



```java
ByteBuffer.allocate(1024);//HeapByteBuffer，堆内内存
ByteBuffer.allocateDirect(1024);//DirectByteBuffer,分配的是堆外内存，可以通过-XX:MaxDirectMemorySize来指定最大的堆外内存

        RandomAccessFile randomAccessFile = new RandomAccessFile("1.txt","rw");
        FileChannel channel = randomAccessFile.getChannel();

        /**
         * 参数一：使用的读写模式
         * 参数二：可以直接修改的起始位置
         * 参数三：映射到内存的大小，单位是字节
         */
		//实际上返回的实例还是DirectByteBuffer，只不过这时候映射的地址是内核中的pagecache,MappedByteBuffer对象还是在堆中
		//MappedByteBuffer使用虚拟内存，因此分配(map)的内存大小不受JVM的-Xmx参数限制，但是也是有大小限制的。
        MappedByteBuffer map = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);

        byte b = map.get();
        System.out.println("使用MappedByteBuffer,b="+b);
        map.put(0, (byte) 'a');
```

### 零拷贝和异步IO

参考文档[https://mp.weixin.qq.com/s/pT1rlPmjDFsfBeFWwdln9A](https://mp.weixin.qq.com/s/pT1rlPmjDFsfBeFWwdln9A)



## 该技术是什么？

Netty 是基于NIO开发的用于网络编程的框架，用于开发服务器。

## 该技术有什么特点(使用注意)：

出栈事件和入栈事件，对应client-->server,server-->client  why????总感觉pipeline中的handler处理过程不是这样的



#### Netty编解码器

> netty 提供了一些编解码器，例如StringEncoder,StringDecoder,ObjectEncoder,ObjectDecoder
>
> 可以对字符串和java对象进行编码和解码
>
> ObjectEncoder 仍然使用的是jdk底层序列化技术，效率低，而且无法跨语言(只能是java客户端，服务端来解析)，序列化后体积庞大，是二进制编码的5倍多
>
> 推荐新的解决方案Google的Protobuf

Protobuf 适合做数据存储或 **RPC数据交换格式**
Protobuf 以message的方式来管理数据的，跨平台，跨语言，支持不同语言的客户端和服务端（C++，JAVA,Python等）

目前公司数据交换方法 http+json转型tcp+protobuf

[Protobuf 文档](https://developers.google.com/protocol-buffers/)

[protobuf编译器下载](https://github.com/protocolbuffers/protobuf/blob/master/src/README.md)

> 注意使用编译器将.proto文件编译成对应语言的文件，如.java

对应语言需要导入相关包，如java

```
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>3.11.0</version>
        </dependency>
```



protobuf 大致流程

![image-20201008223016447](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20201008223030.png)

#### Handler链调用机制

![image-20201011205752788](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20201011205759.png)

**不论是解码器handler还是编码器handler即接收到的消息类型必须与待处理的消息类型一致，否则该handler不会被执行**

**在解码器进行数据解码时，需要判断缓冲区(ByteBuf)的数据是否足够，否则接收到的结果和期望结果可能不一致**

其它的一些解码器

![image-20201011205946934](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20201011205947.png)

> ReplayingDecoder传递一个专门的ByteBuf实现，当缓冲区中没有足够的数据时，这个实现会抛出某种类型的错误，假设在调用`buf.readInt()`时，缓冲区中有4个或更多字节。如果缓冲区中确实有4个字节，它将像您期望的那样返回整数报头。否则，将引发错误并将控制返回到ReplayingDecoder。如果ReplayingDecoder捕捉到错误，那么它会将缓冲区的readerIndex倒回“初始”位置(即缓冲区的开始)，并在缓冲区接收到更多数据时再次调用`decode(..)`方法。
>
> 请注意，ReplayingDecoder总是抛出相同的缓存错误实例，以避免每次抛出时创建新错误并填充其堆栈跟踪的开销。
>
> 
>
> 作者：寅务
> 链接：https://www.jianshu.com/p/9cc9ee3e7ddc
> 来源：简书
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



![image-20201011210137648](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20201011210137.png)



#### TCP 粘包和拆包

解决方案：自定义协议包+编解码器解决，

关键是服务端每次读取数据长度的问题，解决这个问题就能解决粘包和拆包

#### 部分源码剖析



## 该技术怎么使用。demo

## 该技术什么时候用？test。

手写RPC