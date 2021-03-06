# java代码精简

## 1、去重转化

```java
        Set<String> teacherIds=new HashSet<>();
        List<EduTeacher> teacherList=new ArrayList<>();
        List<EduTeacher> resList=new ArrayList<>();
        //避免使用set的contains判断是否已经存在
        for (EduTeacher e:teacherList){
            if (teacherIds.add(e.getId())) {
                resList.add(e);
            }
        }
```

## 2、Map的computeIfAbsent方法

```java
        //computeIfAbsent保证map获取的对象不为null,避免不必要的判null
        Map<String,List<EduTeacher>> map=new HashMap<>();
        List<EduTeacher> teacherList=new ArrayList<>();
        for (EduTeacher e:teacherList){
            map.computeIfAbsent(e.getId(),key->new ArrayList<>()).add(e);
        }
```

## 3、避免空值判断

```java
//使用的spring中的包

//        CollectionUtils.isEmpty();集合
//        StringUtils.isEmpty();字符串
//        Objects.isNull();普通对象
```

## 4、普通的对象拷贝，浅拷贝

```java
//spring的包
//属性名相同才能拷贝，针对很普通的POJO之间的拷贝很方便，非深拷贝
BeanUtils.copyProperties();


```

## 5、简化异常

```java
        if (Objects.isNull(list)) {
            throw new IllegalArgumentException("list null");
        }
		//======下面这种简化的判断null=====
        Assert.notNull(list,"list null");
```

## 6、多个if else 可以考虑定义map常量

```java
 static final Map map=ImmutableMap.of(...);
//
public void fun(){
    //code...
}
```

## 7、数组

```java
Arrays.fill();//方法
//相当于for循环赋值

//list 转成 int数组
int[] arr = list.stream().mapToInt(Integer::valueOf).toArray();

//int数组 转list
Arrays.asList()//List不包括add,remove方法，返回的ArrayList对象是Arrays自己的内部类
List list = new ArrayList<>(Arrays.asList("a", "b", "c"));//这种方法对于数组中的元素是int是不行的，int不是类，如果是二维数组中的每个元素是int[]，可以所使用，int[]是类
//可行
List<Integer> list = Arrays.stream(arr).boxed().collect(Collectors.toList());
```

