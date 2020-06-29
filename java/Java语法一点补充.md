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

