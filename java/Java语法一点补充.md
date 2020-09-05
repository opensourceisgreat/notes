## 匿名内部类初始化Map，List

案例场景：

```java
List<Integer> list = new ArrayList<>(){
    {
        add(3);
        add(4);
    }
}
Map<String,String> map = new HashMap<>(){
    {
        put("a","b");
        put("c","d");
    }
}
```

原理分析：

先创建了匿名内部类，外面的括号可以理解就是重新继承了一个类并且根据该类创建了对象，里面的括号理解为非静态代码块（构造代码块），初始化对象时，会先执行父类构造函数，再执行构造代码块。所以，该构造代码块也可以看成是匿名内部类的构造器。

构造代码块和构造函数的执行顺序：

父类People和子类Customer:

```java
public class People {
    private String name;

    public People() {
        System.out.println("父类构造函数");
    }
    {
        System.out.println("父类的构造代码块");
    }
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

public class Customer extends People{
    public Customer() {
        System.out.println("子类构造函数");
    }
    {
        System.out.println("构造代码块");
    }
}
```

测试类：

```java
public class MainTest{
    public static void main(String[] args) {
        People people = new People(){
            {
                setName("liang");

                System.out.println("匿名内部类的构造代码块");
            }
        };
        System.out.println("==============");
        Customer customer = new Customer();
    }
}
```

执行结果：

```java
父类的构造代码块
父类构造函数
匿名内部类的构造代码块
==============
父类的构造代码块
父类构造函数
构造代码块
子类构造函数
```

从上述测试用例可以看出来，匿名内部类和普通类在创建对象时，都是先执行父类构造代码块，再执行父类构造函数，紧接着执行自己的构造代码块，普通类最后再执行自己的构造函数。注意：==创建子类对象时不会创建父类对象。==参考文档：[java中，创建子类对象时，父类对象会也被一起创建么？](https://blog.csdn.net/lylovelzf/article/details/79417101)

## 匿名内部类（方法中的类）中的变量

参考文档：[为什么匿名内部类和局部内部类只能访问final变量](https://blog.csdn.net/weixin_30287169/article/details/96543710)

> 5)  设方法f被调用,从而在它的调用栈中生成了变量i,此时产生了一个局部内部类对象inner_object,它访问了该局部变量i .当方法f()运行结束后,局部变量i就已死亡了,不存在了.但:局部内部类对象inner_object还可能  一直存在(只能没有人再引用该对象时,它才会死亡),它不会随着方法f()运行结束死亡.这时:出现了一个"荒唐"结果:局部内部类对象 inner_object要访问一个已不存在的局部变量i!
>
>
> 6)  如何才能实现?当变量是final时,通过将final局部变量"复制"一份,复制品直接作为局部内部中的数据成员.这样:当局部内部类访问局部变量 时,其实真正访问的是这个局部变量的"复制品"(即:这个复制品就代表了那个局部变量).因此:当运行栈中的真正的局部变量死亡时,局部内部类对象仍可以 访问局部变量(其实访问的是"复制品"),给人的感觉:好像是局部变量的"生命期"延长了.

没有显示写上 final 修饰变量，编译器在编译后会给变量加上final

## [Java加载资源文件的两种方法](https://www.cnblogs.com/davenkin/archive/2012/04/08/java-load-resources.html)