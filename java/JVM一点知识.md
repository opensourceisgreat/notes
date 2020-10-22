### 参考资料

[内存模型](https://www.processon.com/view/5ea7a1b9e401fd21c196eb17)

[jvm文档](https://www.cnblogs.com/yanl55555/category/1686360.html)

[JVM从入门到精通](https://blog.csdn.net/sinat_42483341/category_10210903.html)

### GC日志信息解释

![](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200619194958.png)

### 类加载

Class对象知识补充：

加载类时会生成Class对象作为访问方法区中类信息的数据访问入口。

生成实例对象时，该对象的对象头会存储指向类元数据的指针，该指针指向JVM中方法区中的类信息。

对象getClass()方法获取的就是Class对象。

### 类构造器

> <clinit>，类构造器方法，在jvm第一次加载class文件时调用， 因为是类级别的，所以只加载一次， 是编译器自动收集类中所有类变量（static修饰的变量）和静态语句块（static{}），中的语句合并产生的， 编译器收集的顺序，是由程序员在写在源文件中的代码的顺序决定的。也就是说语句块和static变量对同一变量初始化时，写代码时谁在后面，谁就是最终初始化的值。
> 
>没有相关static，就没有类构造器方法<clinit>

### classpath

参考文档：

[SpringBoot Jar包启动原理](https://www.jianshu.com/p/c95cac2c2e7a)

[What is a classpath and how do I set it?](https://stackoverflow.com/questions/2396493/what-is-a-classpath-and-how-do-i-set-it/2396759#2396759)

命令 java -cp 等价 java -classpath
普通启动 Java 类通过上述命令来指定 classpath，让JVM知道从哪里来找类

java -jar 

官方说明：

> When you use this option, the JAR file is the source of all user classes, and other user class path settings are ignored.
> 使用此选项时，JAR 文件是所有用户类的源，而其他用户类路径设置将被忽略

通过运行打包后的 springboot 的 jar 包为例  
java -jar myClass.jar
执行该命令时，会用到目录 META-INF\MANIFEST.MF 文件，在该文件中，有一个叫 Main－Class 的参数，它说明了 java -jar 命令执行的类，启动 JarLauncher 后会告诉 JVM classpath 有哪些。
文件内容：
Manifest-Version: 1.0
Implementation-Title: xueyuan_eureka
Implementation-Version: 0.0.1-SNAPSHOT
Start-Class: com.whu.edu.eureka.EurekaServerApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Build-Jdk-Spec: 1.8
Spring-Boot-Version: 2.1.10.RELEASE
Created-By: Maven Archiver 3.4.0
Main-Class: org.springframework.boot.loader.JarLauncher

### 四种引用

强、软、弱、虚

### 类加载过程

![image-20200905153553662](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200905153600.png)

![image-20200905160624535](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200905160624.png)



**类加载器**

![image-20200905161415189](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200905161415.png)

![image-20200905161607866](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200905161608.png)

![image-20200905161650728](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200905161650.png)

### JVM 调优

```
系统CPU经常100%，如何调优？(面试高频)
    CPU100%那么一定有线程在占用系统资源，
    找出哪个进程cpu高（top）
    该进程中的哪个线程cpu高（top -Hp）
    导出该线程的堆栈 (jstack)
    查找哪个方法（栈帧）消耗时间 (jstack)
    工作线程占比高 | 垃圾回收线程占比高


系统内存飙高，如何查找问题？（面试高频）
导出堆内存 (jmap)
分析 (jhat jvisualvm mat jprofiler … )
如何监控JVM
jstat jvisualvm jprofiler arthas top…
```

#### 调优，从规划开始

- 调优，从业务场景开始，没有业务场景的调优都是耍流氓

- 无监控（压力测试，能看到结果），不调优（可以调整业务逻辑）

- 步骤：

  1. 熟悉业务场景（没有最好的垃圾回收器，只有最合适的垃圾回收器）

     1. 响应时间、停顿时间 [CMS G1 ZGC] （需要给用户作响应）
     2. 吞吐量 = 用户时间 /( 用户时间 + GC时间) [PS]

  2. 选择回收器组合

  3. 计算内存需求（没有一定之规，是经验值。 1.5G -> 16G，突然卡顿了，为啥？）

  4. 选定CPU（预算能买到的，当然是越高越好，CPU多核，可以多线程运行呀）

  5. 设定年代大小、升级年龄

  6. 设定

     日志参数，这是Java虚拟机的参数，也可以在Tomcat里面配置，貌似是在叫catalinaoptions里面指定java日志的参数。

     1. `-Xloggc:/opt/xxx/logs/xxx-xxx-gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause`
        5个日志文件，循环产生。生产环境中的日志参数一般这么设置。
        `%t`是生成时间的意思。
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628151904896.png)
     2. 或者每天产生一个日志文件

  7. 观察日志情况

[jhat分析dump文件](https://www.cnblogs.com/baihuitestsoftware/articles/6406271.html)