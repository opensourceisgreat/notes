#### String 的存储分析，JDK1.8

常量池内存分布图：

![](https://my-image-blog.oss-cn-beijing.aliyuncs.com/img/20200613200257.png)

**运行时常量池(Runtime Constant Pool)**

> Runtime constant pool is per-class runtime representation of constant pool in a class. It contains class runtime constants and static methods. Runtime constant pool is part of the method area.

我的理解就是，加载 class 文件到 jvm 中后，Class文件常量池就变成了运行时常量池，类似程序和进程的关系。

在**运行期间也可将新的常量放入运行时常量池**中，比如String的intern方法。

**字符串常量池(String pool):**

JDK1.8 hotspot移除了永久代用元空间(Metaspace)取而代之, 这时候**字符串常量池在堆, 运行时常量池在方法区**

字符串常量池中同时存在字符串常量和字符串引用。直接赋值和用字符串调用String构造函数都可能导致常量池中生成字符串常量;而intern()方法会尝试将堆中对象的引用放入字符串常量池。

intern()方法说明

> ```java
> * When the intern method is invoked, if the pool already contains a    //the pool ==> A pool of strings
> * string equal to this {@code String} object as determined by
> * the {@link #equals(Object)} method, then the string from the pool is
> * returned. Otherwise, this {@code String} object is added to the
> * pool and a reference to this {@code String} object is returned.
> ```

**Class文件常量池(静态常量池)：**

Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池表（用于存放编译期生成的各种字面量和符号引用）

字面量：可以理解为字面意思的常量。比如，字符串字面量："abc"；整型字面量：123。比如：`int a = 123`; `a`是变量，`123`是字面量

符号引用：可以是任意类型的字面量。只要能无歧义的定位到目标。在编译期间由于暂时不知道类的直接引用，因此先使用符号引用代替。最终还是会转换为直接引用访问目标。

直接引用：就是直接指向地址值的引用。

在JVM的类加载过程中，在解析阶段，Java虚拟机会把类的二级制数据中的符号引用替换为直接引用。

**方法区（Method Area）的实现：**

1. 元空间（MetaSpace）占用直接内存

2. 永久代（jdk1.8中被元空间取代）

   在方法区中，存储了**每个类的信息（包括类的名称、方法信息、字段信息）、静态变量、常量以及编译器编译后的代码等**。



**具体案例分析字符串的存储：**

```java
    public static void main(String[] args) {
        String s1="aaa";     //加载该类时，在string pool中创建“aaa”对象
        String s2="bbb"+"ccc"; //通过反汇编代码看出 JVM自动优化的字符串相加，直接在string pool中创建“bbbccc”对象
        //创建StringBuilder对象 ==>String s3 = new StringBuilder().append(s1).append("bbb").toString();
        String s3=s1+"bbb"; //在string pool中创建“bbb”对象
        String s4=new String("kkk");//在string pool中创建“kkk”对象，堆中创建String对象
        String s5="kkk";//s5直接指向string pool中的“kkk”对象
    }
```

上述代码反汇编结果：

指令说明：

1. invokevirtual 用于调用实例的方法
2. invokespecial 用于调用特殊处理方法，比如构造函数、私有方法、父类方法
3. ldc 将一个常量加载到操作数栈
4. astore 将栈顶引用型数值存入指定本地变量；astore_i 将栈顶引用型数值存入第i+1个本地变量
5. aload 将指定的引用类型本地变量推送至栈顶；aload_i将第i+1个引用类型本地变量推送至栈顶
6. dup 复制栈顶数值(数值不能是long或double类型的)并将复制值压入栈顶
7. new  创建对象

```java
public class pers.liang.javatest.maptest.MapTest {
  public pers.liang.javatest.maptest.MapTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String aaa
       2: astore_1
       3: ldc           #3                  // String bbbccc
       5: astore_2
       6: new           #4                  // class java/lang/StringBuilder
       9: dup
      10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
      13: aload_1
      14: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      17: ldc           #7                  // String bbb
      19: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      25: astore_3
      26: new           #9                  // class java/lang/String
      29: dup
      30: ldc           #10                 // String kkk
      32: invokespecial #11                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
      35: astore        4
      37: ldc           #10                 // String kkk
      39: astore        5
      41: return
}

```

加载Class文件时，在string pool中创建字符串字面量的对象，当然不包括这样的的字面量`"aaa"+"bbb"`

#### 参考文章

[JDK1.8关于运行时常量池, 字符串常量池的要点](https://blog.csdn.net/q5706503/article/details/84640762)

[一篇与众不同的 String、StringBuilder 和 StringBuffer 详解](https://mp.weixin.qq.com/s/Y1_ezzXusn9TJW_O5dL0xg)

[JVM内存区域划分](https://blog.csdn.net/q5706503/article/details/84614158)