### GC日志信息解释

![](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200619194958.png)

### 类加载

Class对象知识补充：

加载类时会生成Class对象作为访问方法区中类信息的数据访问入口。

生成实例对象时，该对象的对象头会存储指向类元数据的指针，该指针指向JVM中方法区中的类信息。

对象getClass()方法获取的就是Class对象。

### 类构造器

> 
> <clinit>，类构造器方法，在jvm第一次加载class文件时调用， 因为是类级别的，所以只加载一次， 是编译器自动收集类中所有类变量（static修饰的变量）和静态语句块（static{}），中的语句合并产生的， 编译器收集的顺序，是由程序员在写在源文件中的代码的顺序决定的。
>

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