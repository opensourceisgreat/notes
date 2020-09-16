[toc]


##### 写代码：
1. 明确需求。我要做什么？
2. 分析思路。我要怎么做？1,2,3。
3. 确定步骤。每一个思路部分用到哪些语句，方法，和对象。
4. 代码实现。用具体的java语言代码把思路体现出来。

 

##### 学习新技术的四点：
1. 该技术是什么？
2. 该技术有什么特点(使用注意)：
3. 该技术怎么使用。demo
4. 该技术什么时候用？test。

[Java基础总结](https://www.cnblogs.com/In-order-to-tomorrow/p/3652315.html?utm_source=tuicool&utm_medium=referral)

##### 注意不需要看的内容

动态编译，脚本引擎，字节码，类加载机制（工作用到再说）手写sorm框架（学有余力）


##### 难点待解决
- [ ] 垃圾回收的相关内容
- [ ] [暂时不理解](#question1)


##### 目标
建立知识体系

---

# java基础笔记记录

### JavaSE,JavaEE,JavaME关系

![image](https://note.youdao.com/yws/api/personal/file/39DF3BE4586542FC80078A080E846B1B?method=download&shareKey=0956858af044f64f055579d8d1f44072)

安卓开发和JavaME开发是不同的内容



```
graph LR
    A[*.java源文件] -->|java编译器| B(*.class字节码文件)
    B --> C[类装载器]
    subgraph JRE
        C --> D[字节码校验器]
        D --> E[解释器]
    end
    E --> F[系统平台]

```

### Java跨平台机制

```
    graph LR
        A[*.java源文件] -->|java编译器| B(*.class字节码文件)
        B -->C{编译执行}
        C -->D[JVM for Unix]
        C -->E[JVM for Windows]
        C -->F[JVM for Other]
        

```




### jdk,jre,jvm关系

```
graph TB
    subgraph 
        JDK
        subgraph 
            JRE
            subgraph 
                JVM
            end
        end
    end
```

---

成员变量会自动初始化



### Java数据类型

```
graph LR
    A[数据类型]-->B[基本数据类型]
    A-->C[引用数据类型&#40占用4字节&#41]
    B-->D[数值型]
    B-->字符型&#40char,2个字节&#41
    B-->布尔型&#40boolean,1位&#41
    D-->整数类型&#40byte,short,int,long&#41占1,2,4,8字节
    D-->浮点类型&#40float,double&#41占4,8字节
    C-->类
    C-->接口
    C-->数组

```

### Java整型常量表示
八进制：以0开头，例如015<br>
十六进制：0x开头，例如0x15<br>
二进制:0b开头，例如0b1101<br>
整形常量默认是int型，注明L是long型，例如13333L是long型<br>

注意：==超过int型范围的整型常量需要加上L==

整形常量运算时溢出及时转化成long

```
int a = 1000000000;
long b = 20L * a;
System.out.println(b);
```

### Java浮点型常量

表示方法分为两种：十进制数和科学计数法（314E-2即3.14）<br>
没有后缀f的浮点数值默认为double类型

==浮点数不准确，不要进行比较，计算时也可能不准确==

java.math包的BigInteger和BigDecimal分别可以实现任意精度的整数和浮点数，浮点数用BigDecimal进行比较和计算

### boolean类型

==没有0,1代表false,true的用法==

余数符号和左边操作数相同，例如9%-5=4


### 运算

移位运算<<,>>的乘除2的效率更高

"3"+4=34(字符串连接)




### 自动类型转化

容量小的可以转化成容量大的（这里指的是表示的数值范围大小）



### 静态导入

java.lang包中类是可以直接访问的，无需引入包

```
import java.lang.Math.*;//导入Math的所有静态属性
import java.lang.Math.PI;//导入PI属性
```



### 方法重写

>方法名，参数列表相同<br>
<span id="question1">返回值类型和声明异常类型子类小于等于父类</span><br>
访问权限子类大于等于父类


>toString(),equals()是Object中的方法，经常会被重写

### native
>改关键字修饰的方法，非java代码实现


### ==和equals

没重写的equals()与==一样，比较变量的值

### A instanceof B
>A是否是B的实例
### 静态初始化块和super()
>静态初始化块static和构造其中的super()一样从Object类的static(super)开始执行，经过子类到本类，都需要执行。

### 数组

数组内容存储在堆内存，有初始值
```
int[] a=new int[2];//默认0,0
boolean[] b=new boolean[2];//默认false
```

### 抽象类
>抽象类可以定义普通的方法，父类没有实现，子类必须实现，抽象类不能实例化，只能用来被继承<br>
抽象类的意义就是为子类提供统一的模板，子类去实现具体的抽象方法

### 接口
1. 可以继承多个接口，访问修饰符只能默认或者public
2. 属性只有常量，默认public static final修饰
3. 方法只能且默认是public abstract
4. 子类实现接口，必须实现所有方法，方法访问权限只能是public

### 内部类

1. 成员内部类
    1. 非静态内部类，直接访问外部类成员，而外部类不能直接访问内部类成员；不能有静态方法，静态属性和静态初始化块
    2. 静态内部类，可以看作外部类的静态成员
2. 匿名内部类
    1. 适合只使用一次的类
    2. ``` new 父类构造器(参数列表){匿名内部类类体}```
    3. 匿名内部类类体中访问外面的局部变量时，局部变量必须加final（jdk1.7）
3. 局部内部类

```
package com.liang;

/**
 * @author liang
 * Date 2019/4/13-10:57
 */
public class TestInnerClass {
    public static void main(String[] args) {
        Outer.Inner inner=new Outer().new Inner();
        inner.show();
        Outer.Inner2 inner2=new Outer.Inner2();
        inner2.show2();
        new Outer().show();

        test(new A(){
            @Override
            public void fun() {
                System.out.println("匿名");
            }
        });

    }
    public static void test(A a){
        a.fun();
    }
}

class Outer {
    private int i = 1;

    public void show() {
        System.out.println("outer");
        Inner inner = new Inner();
        System.out.println(inner.i);
    }

    class Inner {
        int i=3;
        void show() {
            System.out.println("外部类：i = " + Outer.this.i);
            System.out.println(this.i);

        }

    }

    static class Inner2{
        void show2(){

        }
    }

}

interface A{
    void fun();
}

```

### String类

字符常量放在常量池

```
String str1 = "liang";
String str2 = "liang";
String str3 = new String("liang");
//str1 == str2,true;str1和str2指向常量池中的同一个地方
//str1 == str3,fasle
//一般用equals比较字符串
```

```
String s1 = "hello " + "world";//编译器自动优化等价于s1="hello world"
String s2 = "kfc";
String s3 = "ig";
String s4 = s1 + s3;//无法优化，编译器不知道变量s2,s3存储的是什么
```

>StringBuilder(线程不安全，效率高，一般使用这个),StringBuffer(线程安全，效率低)，两者是可变字符序列(即对原序列修改信息)。<br>
StringBuilder常用方法:reverse(),append(),serChar(),insert(),delete()

>String是不可变字符序列

```
    //注意循环累加字符串时的问题
    //String字符串拼接
    String str = "abcd";
    for (int i = 0; i < 100; i++) {
        //每次循环都会产生值为i的字符串对象和str与i连接后的对象，非常耗费空间
        str=str+i;
    }
    System.out.println("str = " + str);
    //一般使用StringBuilder进行字符串拼接
```

### 数组拷贝

System类中提供arraycopy(...)方法。
删除数组的内容实际上也是自我拷贝的过程，同样适用arraycopy
数组的扩容实际上也是将原数组拷贝到更大数组的过程

### Arrays类，常用工具类java.util.Arrays
Arrays.toString()是Arrays中的静态方法，打印数组，不是重写Object的toString()，参数列表不同。<br>
sort()方法进行排序<br>
binarySearch()二分查找


### 包装类
>八个基本数据类型都有对应的包装类,int-->Integer


```
graph TB
    A[基本数据类型]-->B[包装类]
    A-->C[字符串类型]
    B-->A
    B-->C
    C-->A
    C-->B
    
    
```
```
    //基本数据类型装换成包装类
    Integer a = new Integer(3);
    Integer b = Integer.valueOf(4);
    
    //包装类转换成基本数据类型
    int c = a.intValue();
    
    //数字字符串转换成包装类对象
    Integer d = new Integer("333");
    
    //包装类对象转换成字符串
    String s = d.toString();
    
    //常见常量,int最大值
    System.out.println(Integer.MAX_VALUE);

```

>自动装箱，自动拆箱
```
Integer a = 123;
//编译器自动执行Integer a = Integer.valueOf(123);

int c = a;//自动拆箱，自动执行int c = a.intValue();

//[-128,127]的整数在系统初始化时就有一个缓存数组
//调用valueOf()时，如果数据在此范围会先从缓存中拿出创建好的对象
Integer k1 = Integer.valueOf(-128);
Integer k2 = Integer.valueOf(-128);
//k1 == k2为true,因为有缓存，k1,k2指向的是同一个对象，查看valueOf()源码即可发现

```


### DateFormat抽象类和其实现类SimpleDateFormat
```
    //指定时间对象转换成指定格式的时间字符串
    DateFormat df = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    System.out.println(df.format(d));

    //指定格式的时间字符串转换成时间对象
    DateFormat df1=new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    Date date = df1.parse("2017-10-1 25:33:11");
    System.out.println(date);
```

### Calendar抽象类和实现类GregorianCalendar
>相当于对已经做好的日历进行操作，例如给定某一日期，就能确定其周几

```
        String s="2019-04-02";
        DateFormat df=new SimpleDateFormat("yyyy-MM-dd");
        Date d=df.parse(s);
        //日期类
        Calendar c = new GregorianCalendar();
        c.setTime(d);
        //将日期从2019-04-02变为2019-04-01
        c.set(Calendar.DAY_OF_MONTH,1);
        System.out.println(c.get(Calendar.DAY_OF_MONTH));
        //星期天-星期六<==>1-7
        System.out.println(c.get(Calendar.DAY_OF_WEEK));
        //1-12月<==>0-11
        System.out.println(c.get(Calendar.MONTH));
```

### File类
```
    public static void main(String[] args) throws ParseException, IOException {

        File file = new File("D:\\下载");
        printFiles(file,0);
    }
    /**
     *  递归打印目录树 
     */
    static void printFiles(File file,int level){
        for (int i = 0; i < level; i++) {
            System.out.print("-");
        }
        System.out.println(file.getName());
        if(file.isDirectory()){
            File[] files = file.listFiles();
            for (File f : files) {
                printFiles(f,level+1);
            }
        }
    }
```
### 枚举
>有必要定义一组常量的时候使用枚举，不需要使用枚举的高级特性

>枚举也相当于类，成员相当于常量

```
public class TestData {
    public static void main(String[] args) throws ParseException, IOException {
        Season s = Season.AUTUMN;
        System.out.println("s = " + s);
        switch (s){
            case AUTUMN:
                System.out.println("秋");
                break;
            case SPRING:break;
            case SUMMER:break;
            case WINTER:break;
        }
    }
}
enum Season {
    SPRING, SUMMER, AUTUMN, WINTER
}
```

### 异常
>运行时异常需要我们进行逻辑判断处理

>CheckedException编译时就会报告错，try..catch..finally,不管有没有异常finally必会执行.<br>
try{}里有一个return语句，紧跟在这个try后的finally{}里的code会不会被执行，什么时候被执行，在return前还是后?
答：肯定会执行，finally{}块的代码只有在try{}块中包含遇到System.exit(0)之类的导致Java虚拟机直接退出的语句才会不执行。<br>
当程序执行try{}遇到return时，程序会先执行return语句，但并不会立即返回——也就是把return语句要做的一切事情都准备好，也就是在将要返回、但并未返回的时候，程序把执行流程转去执行finally块，当finally块执行完成后就直接返回刚才return语句已经准备好的结果。

>多个catch分支，子类异常处理分支放在父类异常前面，否则子类异常根本不会执行，因为父类异常就可以接受处理了。偷懒就用Exception就可以接受所有异常

### java泛型
此处讲的比较详细
https://www.cnblogs.com/coprince/p/8603492.html

https://blog.csdn.net/u013066244/article/details/79985348

https://blog.csdn.net/u011848397/article/details/88949550

### Collection

![image](https://note.youdao.com/yws/api/personal/file/CACBAC28FD954772BD82B5B1D8E5ACE7?method=download&shareKey=aa7796595347167165fdb26efbc9e70f)

##### List
List是有序，可重复的容器，每个元素有索引<br>
remove()是移除容器中的引用，对象还是存在的，等系统回收

ArrayList底层是用的数组，查询效率高，增删效率低，线程不安全，但是一般使用它

==ArrayList扩容时直接扩容原大小的一半加原大小==

LinkedList,底层是双向链表，查询效率低，增删效率高，线程不安全

Vector ,底层是用数组实现的List，线程安全，效率低

##### Set
>无序，不可重复，不能加入同样内容的元素，没有索引只能遍历查找

>HashSet底层用HashMap实现

>TreeSet，底层是TreeMap,需要排序，所以对应的类要实现Comparable接口

### Map
>map中键不能重复，若重复（根据equals方法来判断），新的覆盖旧的

==默认初始容量16，负载因子0.75,可以自定义，存储量达到12就会扩容，直接2的5次方，下次扩容都是2的6次方，以此类推==


>==HashMap==,底层采用了哈希表(数组+链表)

![image](https://note.youdao.com/yws/api/personal/file/81A17F036037439EA631B1BD0B2C5B2B?method=download&shareKey=c4c3078a5ce8b3ed158a0ba3ec740ae3)

![image](https://note.youdao.com/yws/api/personal/file/66151DD5F4BB463698A1F6E7DC76DED3?method=download&shareKey=1d565c58329de7a8125998a03cca4574)


>取余获取hash值，改进后的算法，hash值=hashcode&(数组长度（必须是2的整数幂）-1)相当于取余效果，效率更高，当然jdk源码还进行了其他移位操作让数据尽可能散开；<br>
jdk8中当链表长度大于8时，链表会转成==红黑树==，大大提高效率

 ==问题:== 当数组扩容后，通过key值（没扩容前就存放进去的值）计算出来的hash值和原来的hash值应该会不同，算法是如何解决的？实际上就是resize()方法，扩容后会对将旧数组重中的数据在新数组总重新计算hash值散列一遍

 ==注意上述的hash值和hashcode是不一样的，hash值指的是该数在数组中存放的索引，hashcode是通过key算的==

>HashTable和HashMap几乎一样，但是前者线程安全，效率低，不允许Key或Value为空

>TreeMap，红黑二叉树的典型实现，需要对map排序时使用它，（按key排序），自定义排序时，自己写的类需要实现comparable接口，重写其中的compareTo()方法

```
JAVA 两个对象不同为什么他们的hashcode有可能相同
  hashCode是所有java对象的固有方法，如果不重载的话，返回的实际上是该对象在jvm的堆上的内存地址，而不同对象的内存地址肯定不同，所以这个hashCode也就肯定不同了。如果重载了的话，由于采用的算法的问题，有可能导致两个不同对象的hashCode相同。

而且，还需要注意一下两点：

1）hashCode和equals两个方法是有语义关联的，它们需要满足：

A.equals(B)==true --> A.hashCode()==B.hashCode()

因此重载其中一个方法时也需要将另一个也重载。

2）hashCode的重载实现需要满足不变性，即一个object的hashCode不能前一会是1，过一会就变成2了。hashCode的重载实现最好依赖于对象中的final属性，从而在对象初始化构造后就不再变化。一方面是jvm便于代码优化，可以缓存这个hashCode；另一方面，在使用hashMap或hashSet的场景中，如果使用的key的hashCode会变化，将会导致bug，比如放进去时key.hashCode()=1，等到要取出来时key.hashCode()=2了，就会取不出来原先的数据。这个可以写一个简单的代码自己验证一下

哈希表是结合了直接寻址和链式寻址两种方式，所需要的就是将需要加入哈希表的数据首先计算哈希值，其实就是预先分个组，然后再将数据挂到分组后的链表后面，随着添加的数据越来越多，分组链上会挂接更多的数据，同一个分组链上的数据必定具有相同的哈希值，java中的hash函数返回的是int类型的，也就是说，最多允许存在2^32个分组，也是有限的，所以出现相同的哈希码就不稀奇。
```

### Iterator迭代器遍历容器元素<List,Set,Map>

![image](https://note.youdao.com/yws/api/personal/file/0E8F3D0115BC40DDB37723AA85E4A295?method=download&shareKey=2d2a1efcaab2df7bec7a1a6ba640a257)
![image](https://note.youdao.com/yws/api/personal/file/899B75A455FE48DC90F7044CEA6C6B88?method=download&shareKey=6f3330828c8bc1b8e080dd6f5187c93c)
![image](https://note.youdao.com/yws/api/personal/file/2932BE466B5F4E7C8C8D50348D531D6B?method=download&shareKey=5c0e7ecae3aa3cb62023aaf7c46edd81)


### Collections工具类，不是Collection接口

![image](https://note.youdao.com/yws/api/personal/file/504A015666574532AEF2A345A2AA376B?method=download&shareKey=a7b1eb6f6fbd722b9e26d609a54dc8f4)


### 注解
>可以被其他程序所理解（如，编译器），注释是不行的

>内置注解<br>
@Override<br>
重写父类方法<br>
@Deprecated<br>
表示不建议使用该方法<br>
@SuppressWarnings<br>
抑制编译器的警告，需要添加参数使用

```
//元注解负责注解其他注解
//元注解Target，指定自定义的注解使用位置
@Target(@Target(value = {ElementType.TYPE}))
//该元注解表明注解的有效时段{SOURCE(源文件有效),CLASS(class文件有效),RUNTIME(运行时有效)}
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    //参数类型名 参数名() [default 默认值]，不加default时，使用该注解时必须添加对应参数
    String name() default "";
    //默认值为-1，代表没有添加该参数时代表不存在该参数。
    int id() default -1;
    String[] people() default {"xiaohong","xiaoming"};
    //只有一个参数时，一般用value作为名字
    String value();
}


```

### 反射

>就是在程序运行过程中，每一个加载的类都有一个Class对象，通过它的api操作这个类的所有结构

>反射执行效率低，禁止访问安全检查可以提高效率


>反射操作泛型，具体用到再直接使用代码

```

假定Iphone类
//获取类模子,下面是常用的，其他方法不推荐
Class clazz= Class.forName("包名.类名")
//创建对象
Iphone i=(Iphone) clazz.getConstructor().newInstance()

上面由于获取模子时使用字符串，可以很好的解耦，不会再编译时报错，就是字符串所表示的类名，不需要保证编译时就存在，运行时存在就可以了
反射强调的是动态性
创建对象时，左边一般是接口，所有创建的模子一般是某个子类，这是jdk9中建议使用的
反射强调的是动态性
```


Student.java
```
package com.liang;

/**
 * @author liang
 * Date 2019/4/18-14:47
 */
@WhpuTable("table_student")
public class Student {
    @WhpuField(columnName = "id",type = "int",length = 10)
    private int id;
    @WhpuField(columnName = "whpuname",type = "varchar",length = 20)
    private String name;
    @WhpuField(columnName = "age",type = "int",length = 10)
    private int age;

    public Student() {
    }

    public Student(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```

WhpuField.java
```
package com.liang;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @author liang
 * Date 2019/4/18-14:53
 */
@Target(value = {ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface WhpuField {
    String columnName();
    String type();
    int length();
}

```

WhpuTable.java
```
package com.liang;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;


/**
 * @author liang
 * Date 2019/4/18-14:43
 */
@Target(value = {ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface WhpuTable {
    String value();
}

```

TestReflect.java
```
package com.liang;


import java.lang.annotation.Annotation;
import java.lang.reflect.Field;

/**
 * 利用反射读取注解信息
 *
 * @author liang
 * Date 2019/4/18-15:14
 */
public class TestReflect {
    public static void main(String[] args) {
        try {
            String path = "com.liang.Student";
            Class clazz = Class.forName(path);
            //获取类的所有注解信息（仅仅是类的，而不是属性，方法）
            Annotation[] annotation = clazz.getAnnotations();
            for (Annotation a : annotation) {
                System.out.println(a);
            }
            //获取类的指定注解的信息
            WhpuTable a = (WhpuTable) clazz.getAnnotation(WhpuTable.class);
            System.out.println(a.value());
            //获取类的属性注解信息
            Field f = clazz.getDeclaredField("name");
            WhpuField wf = f.getAnnotation(WhpuField.class);
            System.out.println(wf.columnName() + "---" + wf.type() + "---" + wf.length());

            //上述语句获得的值可以拼接成sql语句

            Class clazz1 = Class.forName(path);
            //两个对象hashcode相同，说明是同一个对象，类只加载一次，只对应一个Class对象
            System.out.println(clazz.hashCode());
            System.out.println(clazz1.hashCode());
            //获取Class对象的方式
            Class strClazz = String.class;
            Class strClazz2 = path.getClass();
            Class intClazz = int.class;

            int[] arr1 = new int[2];
            int[] arr2 = new int[4];
            //长度不同但是获取到的是同一个类对象
            System.out.println(arr1.getClass().hashCode() == arr2.getClass().hashCode());
            int[][] arr3 = new int[12][];
            //false
            System.out.println(arr1.getClass().hashCode() == arr3.getClass().hashCode());


        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

Demo.java
```
package com.liang;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * 利用反射操作类
 * @author liang
 * Date 2019/4/18-16:32
 */
public class Demo {

    public static void main(String[] args) {
        String path = "com.liang.Student";
        try {
            Class<Student> clazz = (Class<Student>) Class.forName(path);
            //获取类名
            //包名+类名
            System.out.println(clazz.getName());
            //类名
            System.out.println(clazz.getSimpleName());

            //获取属性信息
            //只能获得public属性
//            Field[] fields=clazz.getFields();
            //返回所有属性
            Field[] fields=clazz.getDeclaredFields();
            //获取指定属性
            Field f=clazz.getDeclaredField("name");



            //获取方法信息
            Method[] methods=clazz.getDeclaredMethods();
            Method method=clazz.getMethod("getId",null);
            Method method1=clazz.getMethod("setId",int.class);
            for (Method m : methods) {
                System.out.println("方法:"+m);

            }

            //获取构造器信息
            Constructor[] constructors=clazz.getDeclaredConstructors();
            Constructor c1=clazz.getDeclaredConstructor(null);
            Constructor<Student> c2=clazz.getDeclaredConstructor(int.class,String.class,int.class);

            for (Constructor c : constructors) {
                System.out.println("构造器："+c);
            }
            //通过反射api调用构造方法，创建对象
            //调用无参构造方法
            Student s1= (Student) c1.newInstance();
            //Class定义了泛型，所以不需要转化
            Student s2= clazz.newInstance();
            //同理Constructor
            Student s3= c2.newInstance(10,"xiao",10);

            //调用普通方法
            //方法一：利用上述创建的对象即可
            //方法二:利用上述method1,等价于s2.setId(12);
            method1.invoke(s2,12);

            //操作属性,不能访问私有属性
            //不做安全检查，可以访问了，私有方法也可以这样做
            f.setAccessible(true);
            f.set(s2,"keng");
            System.out.println(f.get(s2));

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```



### 注意
javabean写类的时候，养成习惯加入空构造器，框架中利用反射构造五参数对象时，需要利用它


### 动态代理(dynamic proxy)

>动态生成代理类和对象


Star.java
```
package com.dynamicProxy;

/**
 * @author liang
 * Date 2019/4/19-9:44
 */
public interface Star {

    /**
     * 订票
     */
    void bookTicket();
    /**
     * 唱歌
     */
    void sing();
    /**
     * 收钱
     */
    void collectMoney();
}

```

RealStar.java
```
package com.dynamicProxy;

/**
 * @author liang
 * Date 2019/4/19-9:44
 */
public class RealStar implements Star{

    @Override
    public void bookTicket() {
        System.out.println("真实角色订票");
    }

    @Override
    public void sing() {
        System.out.println("真实角色唱歌");
    }

    @Override
    public void collectMoney() {
        System.out.println("真实角色收钱");
    }
}

```
StarHandler.java
```
package com.dynamicProxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author liang
 * Date 2019/4/19-9:45
 */
public class StarHandler implements InvocationHandler {
    Star realStar;

    public StarHandler(Star realStar) {
        super();
        this.realStar = realStar;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //所有代理类对象都来这里报道
        Object object = null;
        System.out.println("核心方法执行前");
        System.out.println("订票");

        if (method.getName().equals("sing")) {
            object = method.invoke(realStar, args);
        }
        System.out.println("核心方法执行后");
        System.out.println("收款");
        return object;
    }
}

```


Client.java
```
package com.dynamicProxy;

import java.lang.reflect.Proxy;

/**
 * @author liang
 * Date 2019/4/19-9:49
 */
public class Client {
    public static void main(String[] args) {
        Star realStar=new RealStar();
        StarHandler handler=new StarHandler(realStar);
        Star proxy=(Star) Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(),new Class[]{Star.class},handler);

        proxy.sing();


    }
}

```

>实际上生成代理类对象后，它进行了如下调用,代理类对象的每一个方法都调用handler.invoke()

ProxyStar.java
```
package com.dynamicProxy;

/**
 * @author liang
 * Date 2019/4/19-10:17
 */
public class ProxyStar implements Star{
    @Override
    public void bookTicket() {
        handler.invoke(this,当前方法,args);
    }

    @Override
    public void sing() {
        handler.invoke(this,当前方法,args);
    }

    @Override
    public void collectMoney() {
        handler.invoke(this,当前方法,args);
    }
}

```


### 日后解决

```
        //预留问题：程序执行时分配的两个空间，为何会根据对象使用顺序发生地址变化
        //目前认为hashCode()根据对象内部地址各个方面信息产生hash值（产生方法也很多种），所以有变化。但是可以确定的是他不是真正的地址，
        //以及知道目前对象真正地址不同就行了，以后再研究
        Student s1 = new Student("liang");
        Student s2 = new Student("lei");

        System.out.println("s1 = " + s1.name + " " + s1.toString());
        System.out.println("s2 = " + s2.name + " " + s2.toString());

```



