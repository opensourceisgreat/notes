# 1、Lambda表达式

符号：`->`箭头操作符

左边：形参列表(param1,param2...)，右边：lambda体，即重写的抽象方法的方法体

==lambda本质：作为函数式接口（只声明了一个抽象方法的接口）的实例==，只针对只有一个方法需要重写的接口



`@FunctionalInterface`注解可以用来检验接口是否为函数式接口

总结：形参列表的类型可省略，只有一个参数时，小括号可省略

lambda体用{}包裹，当只有一条语句时，可省略{},是return语句时，return和{}一起省略

```java
public void test(){
        Comparator<Integer> com=new Comparator<Integer>() {
            public int compare(Integer o1,Integer o2) {
                return Integer.compare(o1,o2);
            }
        };
        int compare1=com.compare(1,2);
        System.out.println(compare1);
        System.out.println("*******************");
        //Lambda
        Comparator<Integer> com1=(o1,o2)->Integer.compare(o1,o2);

        System.out.println(com1.compare(1,2));
```

应用：使用匿名实现类时都可以用lambda表达式来做

==注意：lambda体内外部变量必须是final类型（外部可以不声明，编译时会将lambda体内的外部变量自动加上final类型），总之该变量的值不允许改变==

为什么 Lambda 表达式(匿名类) 不能访问非 final 的局部变量呢？

因为实例变量存在堆中，而局部变量是在栈上分配，Lambda 表达(匿名类) 会在另一个线程中执行。如果在线程中要直接访问一个局部变量，可能线程执行时该局部变量已经被销毁了，而 final 类型的局部变量在 Lambda 表达式(匿名类) 中其实是局部变量的一个拷贝。

上述表明函数式编程天生适合并行运行，保证内部使用的变量不会被其他线程更改

# 2、函数式接口

```java
    /**
     * 四大内置函数式接口
     * 消费型接口 Consumer<T> void accept(T t)
     * 供给型接口 Supplier<T> T get()
     * 函数型接口 Function<T,R> R apply(T t)
     * 断定型接口 Predicate<T> boolean test(T t)
     */
    @Test
    public void test2() {
        getMoney(500, x -> System.out.println(x));
    }

    public void getMoney(double money, Consumer<Double> con) {
        con.accept(money);
    }

    @Test
    public void test3() {
        List<String> list = Arrays.asList("北京", "天津", "南京");
        System.out.println(filterString(list, s -> s.contains("京")));
    }

    //根据给定的规则过滤list
    public List<String> filterString(List<String> list, Predicate<String> pre) {
        ArrayList<String> filterList = new ArrayList<>();
        for (String s : list) {
            if (pre.test(s)) {
                filterList.add(s);
            }
        }
        return filterList;
    }
```

还有一些内置的函数式接口，遇到再积累

==函数式接口中可以有default声明的方法，也就是说接口中可以有默认的实现，default修饰的方法==
```

```

# 3、方法引用与构造器引用

## 3.1方法引用

使用情景：当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用

方法引用的要求：要求接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同(针对三种情况的前两种)

方法引用本质上就是Lambda表达式

使用格式： `类（或者对象）:: 方法名`

三种情况：

`对象 :: 非静态方法（实例方法）`，`类 :: 静态方法`,`类 :: 非静态方法`





```java
//Consumer中的void accept(T t)
//PrintStream中的void println(T t)
Consumer<String> con=str-> System.out.println(str);
con.accept("ss");

//System.out(对象),println(方法)
PrintStream ps=System.out;
Consumer<String> con2=ps::println;
con2.accept("ssf");
//
Consumer<String> con3=System.out::println;
con3.accept("xxxx");


//类::静态方法
//Comparator中的int compare(T t1,T t2)
//Integer中的int compare(T t1,T t2)
Comparator<Integer> com1 = (o1, o2) -> Integer.compare(o1, o2);

System.out.println(com1.compare(1, 2));
System.out.println("****************");

//方法引用
Comparator<Integer> com2 = Integer::compare;
System.out.println(com2.compare(1, 2));

//Function中的R apply(T t)
//Math中的Long round(Double d)
Function<Double,Long> func1=x->Math.round(x);
System.out.println(func1.apply(3.0));
Function<Double,Long> func2=Math::round;

//类::实例方法
//Comparator中的int compare(T t1,T t2)
//String中的int t1.compareTo(t2)
Comparator<String> com=(s1,s2)->s1.compareTo(s2);
System.out.println(com.compare("a","b"));
Comparator<String> com2=String::compareTo;
System.out.println(com2.compare("a","b"));
```

## 3.2构造器引用

和方法引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一致

抽象方法的返回值类型即为构造器所属的类的类型

```java
//Supplier中的T get()
//Employee的空参构造器： Employee()
//构造器引用
Supplier<Employee> sup=()->new Employee();
Supplier<Employee> sup2=Employee :: new;

//Function中的R apply(T)
//Employee的有参构造函数：Employee(Integer id)
Function<Integer,Employee> fun1=id->new Employee(id);
Function<Integer,Employee> fun2=Employee :: new;
```

## 3.3数组引用

把数组看做一个特殊的类，则写法与构造器引用一致

```java
Function<Integer,String[]> func1=length->new String[length];

Function<Integer,String[]> func2=String[] :: new;
String[] arr=func2.apply(10);
```



# 4、强大的Stream API

集合讲数据，Stream讲计算

集合和内存打交道，Stream和CPU打交道

放在sql中的一些操作可以拿到java中来做了，尤其从redis等nosql数据库取出数据后

![a.PNG](https://note.youdao.com/yws/api/personal/file/7EF1F570ED0A4C8D960F15A4E6D9AF72?method=download&shareKey=1561495e31f6ccf54cafe11980620f28)

![b.PNG](https://note.youdao.com/yws/api/personal/file/B9F5F0BAB33A406C89E99D2F8B0463C6?method=download&shareKey=0825cae08a3f75fa442a98f0b0b95eb1)

## 4.1创建Stream

```java
        List<Employee> employees=new ArrayList<Employee>();
        employees.add(new Employee(1,"Tom","男"));
        employees.add(new Employee(2,"Tom1","女"));
        employees.add(new Employee(3,"Tom2","男"));
        //创建Stream方式一：集合
        //返回一个顺序流
        Stream<Employee> stream1 = employees.stream();
        //返回一个并行流
		//并行流就是把内容分割成多个数据块，每个数据块对应一个流，然后用多个线程分别处理每个数据块中的流
        Stream<Employee> employeeStream = employees.parallelStream();



        //创建Stream方式二：数组
        int[] a=new int[]{1,2,3};
        IntStream stream = Arrays.stream(a);

        Employee[] arr1=new Employee[]{new Employee(4,"jack","女"),new Employee(5,"jack1","女")};
        Stream<Employee> stream2 = Arrays.stream(arr1);
        

        //创建Stream方式三：通过Stream的of()
        Stream<Integer> integerStream = Stream.of(1, 2, 3, 4);


        //创建Stream方式四：创建无限流
        //迭代，不加limit中间操作，会导致无限迭代t，本例造偶数
        Stream.iterate(0,t->t+2).limit(10).forEach(System.out::println);
        //生成，造十个随机数
        Stream.generate(Math::random).limit(10).forEach(System.out::println);
```



## 4.2Stream中间操作

### 4.2.1筛选与切片

![c.PNG](https://note.youdao.com/yws/api/personal/file/2524782FE8D7462E8919706686D57633?method=download&shareKey=a2a3bde6e7906127083554258b95bc33)

```java
        stream1.filter(e->e.getId()>2).forEach(System.out::println);
        //stream1结束后，不能在使用，必须用新的
        employees.stream().limit(2).forEach(System.out::println);
        employees.stream().skip(2).forEach(System.out::println);
		employees.stream().distinct().forEach(System.out::println);
```

### 4.2.2映射

![d.PNG](https://note.youdao.com/yws/api/personal/file/42BC1412BA0346C99FF7AB9DE84F5D00?method=download&shareKey=333ab790f48ec6c4ce281fca7234a756)

```java
        List<String> list=Arrays.asList("aa","bb");
        list.stream().map(str->str.toUpperCase()).forEach(System.out::println);
        
        employees.stream().map(e->e.getName()).filter(name->name.length()>3).forEach(System.out::println);
        employees.stream().map(Employee::getName).filter(name->name.length()>3).forEach(System.out::println);

//map,flapMap类似list.add和list.addAll
    public void test3(){
        List<String> list =Arrays.asList("aa","bb");
    Stream<Stream<Character>>streamStream=list.stream().map(AppTest::fromStringToStream);
        streamStream.forEach(s->{
            s.forEach(System.out::println);
        });

        Stream<Character>characterStream=list.stream().flatMap(AppTest::fromStringToStream);
        characterStream.forEach(System.out::println);
    }
    
    public static Stream<Character> fromStringToStream(String str){
        ArrayList<Character> arrayList =new ArrayList<>();
        for (Character c:str.toCharArray()){
            arrayList.add(c);
        }
        return arrayList.stream();
    }
```

### 4.2.3排序

```java
//sorted()----自然排序
        List<Integer> list=Arrays.asList(12,3,2,15);
        list.stream().sorted().forEach(System.out::println);


//sorted(Comparator com)----定制排序 ，针对POJO
employees.stream().sorted((e1,e2)->Integer.compare(e1.getId(),e2.getId())).forEach(System.out::println);
```

## 4.3Stream终止操作

### 4.3.1匹配与查找

![e.PNG](https://note.youdao.com/yws/api/personal/file/FC0D7B43E8414E259ACED69AC51AE43B?method=download&shareKey=1f7fc7465ebc32e8d60b78d578d2f286)

`forEach(Consumer c)`内部迭代

Collection接口需要用户去做迭代，称为外部迭代，就是自己需要指定一个指针去指向每个元素，内部迭代就是帮你做了这件事

注意：

```java
employees.stream().forEach();
//使用的是集合中的遍历操作
employees.forEach();
```



### 4.3.2归约

![f.PNG](https://note.youdao.com/yws/api/personal/file/FDE9A34FB6DE4C7488CB65782A2C07E5?method=download&shareKey=6b0675eb0be26527c061d421c3cc1679)

```java
        List<Integer>  list=Arrays.asList(1,2,3,4,5);
        //T reduce(T identity, BinaryOperator<T> accumulator);
        //T identity初始归约值
        Integer sum=list.stream().reduce(0,Integer::sum);

        //
        List<Employee> employees=new ArrayList<Employee>();
        employees.add(new Employee(1,"Tom","男"));
        employees.add(new Employee(2,"Tom1","女"));
        employees.add(new Employee(3,"Tom2","男"));
        Optional<Integer> sum2 = employees.stream().map(Employee::getId).reduce(Integer::sum);
        Optional<Integer> sum3 = employees.stream().map(Employee::getId).reduce((d1,d2)->d1+d2);
```



### 4.3.3收集

![g.PNG](https://note.youdao.com/yws/api/personal/file/548D3F33017A4BFB853BEDFE2F27026F?method=download&shareKey=f0426a73f7acbd21cebfa2feae09eb2b)

```java
        List<Employee> employees=new ArrayList<Employee>();
        employees.add(new Employee(1,"Tom","男"));
        employees.add(new Employee(2,"Tom1","女"));
        employees.add(new Employee(3,"Tom2","男"));
        List<Employee> collect = employees.stream().filter(e -> e.getId() > 2).collect(Collectors.toList());
```



![h.PNG](https://note.youdao.com/yws/api/personal/file/13E95D92EC5745E98B5D14D3A14DBD97?method=download&shareKey=b8f71a7251c7b7281512579e04795958)

![i.PNG](https://note.youdao.com/yws/api/personal/file/7BC6C8D69468460BBA1A5DA4A4BC0573?method=download&shareKey=0488e9c126f9e56e23a0387789214957)

# 5、Optional类

![j.PNG](https://note.youdao.com/yws/api/personal/file/6B74A294CF024402A2CD86C57C983C95?method=download&shareKey=e8533424779ddb45a260f48afff8ba5b)

```java
//Optional为了在程序中避免空指针异常
//常用方法：ofNullable(T t)
//		orElse(T t)
```

![k.PNG](https://note.youdao.com/yws/api/personal/file/7969B421E2EE4C05900BA21CA7AE524F?method=download&shareKey=4f70b072a64f62aa81cd6a5eaf0b5fd7)

