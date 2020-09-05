## 参考文档

[ssm文档](http://mp.weixin.qq.com/mp/homepage?__biz=Mzg2NTAzMTExNg==&hid=3&sn=456dc4d66f0726730757e319ffdaa23e&scene=18#wechat_redirect)

## Spring 优点

Spring 是一个轻量级的控制反转和面向切面编程的框架

但是，它是“配置地狱”，大量繁琐的配置

## @Autowired 自动注入

先ByType 查找符合的对象，如果有多个对象（这些对象是同一个类产生的），会报错不唯一，

如果`@Autowired` 标注的属性类型是一个接口或者父类，在只有一个该类型的子类或实现类时，会按类型匹配，如果有多个实现类或者子类，会按照属性名匹配对象的id名、别名等

总之，会先ByType (有且只有一个匹配就成功，如果有多个匹配就再ByName匹配)

注解方式不需要set方法，配置文件注入需要set或构造器方法注入

## 目标

学习两个概念：
1. IoC
2. AOP

## 环境说明
导入如下包
```
commons-logging-1.1.1.jar
spring-beans-3.2.18.RELEASE.jar
spring-context-3.2.18.RELEASE.jar
spring-core-3.2.18.RELEASE.jar
spring-expression-3.2.18.RELEASE.jar
```

## IoC
>控制反转，将程序中手动创建对象的控制权交给spring框架

## DI
依赖注入
```
    <!--配置一个bean(对象)-->
    <!--原理：
        1.解析xml文件，获取类名，id，属性
        2.通过反射，用类型创建对象
        3.给创建的对象赋值
    -->
    <bean id="userService" class="com.liang.service.UserServiceImpl">
        <!--依赖注入数据，调用属性的set方法-->
        <property name="name" value="xiaoming"></property>
    </bean>
    
    相当于创建一个对象，并且调用对象的属性name的set方法
```

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--依赖注入的三种方法-->
    <!--第一种：使用构造器,前提是有构造方法可以调用-->
    <bean id="user" class="com.liang.model.User">
        <constructor-arg name="age" value="12"></constructor-arg>
        <constructor-arg name="name" value="xiaoqiang"></constructor-arg>
    </bean>

    <!--通过索引加类型，给构造方法赋值-->
    <bean id="user" class="com.liang.model.User">
        <constructor-arg index="0" value="xiaoqiang" type="java.lang.String"></constructor-arg>
        <constructor-arg index="1" value="22" type="int"></constructor-arg>
    </bean>

    <!--第二种：使用set方法-->
    <bean id="user" class="com.liang.model.User">
        <!--依赖注入数据，调用属性的set方法-->
        <property name="name" value="xiaoming"></property>
        <property name="age" value="12"></property>
    </bean>

    <!--第三种(用的比较少)：使用p命名空间,文件头部添加xmlns:p="http://www.springframework.org/schema/p"-->
    <bean id="user" class="com.liang.model.User" p:age="12" p:gender="男" p:name="xiao"></bean>

</beans>
```

```
    <!--一个对象引入另外一个对象(先创建要引入的对象)
        两种方法：
        1.ref:<property name="address" ref="address"></property>
        2.SpEL:<property name="address" value="#{address}"></property>
    -->
    <bean id="address" class="com.liang.model.Address">
        <property name="name" value="武汉"></property>
    </bean>

    <bean id="user" class="com.liang.model.User">
        <property name="address" value="#{address}"></property>
    </bean>
```
SpEL表达式用到在学习


注入集合
```
<!--List-->
<property name="cars">
    <list>
        <value>...</value>
        <value>...</value>
    </list>
</property>

<!--Set-->
<property name="cars">
    <set>
        <value>...</value>
        <value>...</value>
    </set>
</property>

<!--Map-->
<property name="cars">
    <map>
        <entry key="..." value="..."></entry>
        <entry key="..." value="..."></entry>
        <entry key="..." value="..."></entry>
    </map>
</property>

<!--Properties-->
<property name="myInfoProperties">
    <props>
        <prop key="..."></prop>
        <prop key="..."></prop>
        <prop key="..."></prop>
    </props>
</property>


<!--数组-->

<property name="cars">
    <array>
        <value>...</value>
        <value>...</value>
    </array>
</property>
```

## 注解注入
注解用来取代xml

1. 开启注解功能
```
<!--注意头部-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启注解功能-->
    <context:annotation-config/>

    <!--注解的位置-->
    <context:component-scan base-package="com.liang"/>

    <!--类加上@Component
    相当于<bean class="com.liang.model.Student"></bean>
    -->
    <!--类加上@Component("student")
    相当于<bean id="student" class="com.liang.model.Student"></bean>
    -->
</beans>

//测试代码
public void test() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans3.xml");
    //@Component没有配置id，通过类型获取,也可以通过接口类型，它会自动去找实现类,注解本身要放在实现类
    //Student student= (Student) context.getBean(Student.class);
    //@Component("student"),通过id获取
    //Student student= (Student) context.getBean("student");

}
```

web层，service层，dao层，不用注解时需要手动配置bean和依赖注入，通过三种component注解的变体实现三层之间的调用，因为三层生成对象是有顺序的

```
//dao层

@Repository
public class StudentDaoImpl implements IStudentDao{
    //<bean init-method="" />
    @PostConstruct
    public void myInit(){
        //自定义初始化方法，创建对象后就执行
    }

    @Override
    public void add(Student student) {
        System.out.println("StudentDaoImpl：增加学生:"+student);
    }
    //<bean destroy-method="" />
    @PreDestroy
    public void myDestroy(){
        //对象马上销毁，在销毁前执行
    }
}

//service层
@Service("myService")//指定id，可以不需要
public class StudentServiceImpl implements IStudentService {
    @Autowired//自动注入属性值，不需要手动写set方法,不需要new
    private IStudentDao iStudentDao;


    @Override
    public void add(Student student) {
        System.out.println("service增加学生:"+student);
        iStudentDao.add(student);
    }
}

//web层
@Controller
@Scope("prototype")//指定多例，而非单例
public class AddStudentController {
    
    @Autowired//该注解可以实现自动注入属性值，不需要手写set方法，不需要new，如果是接口就去容器找接口的实现类
    @Qualifier("myService")//根据指定的id注入，可以不需要
    //@Resource(name="myService")等同于上两行，很少用
    private IStudentService iStudentService;

    public void add(Student student){
        System.out.println("controller增加学生："+student);
        iStudentService.add(student);
    }
}

```
==@Autowired放在方法上会给参数自动从容器中找值注入==

## 通过spring创建对象

```
    @Test
    public void test1() {
        //spring容器的3种加载方式
        //第一种（最常用）：ClassPathXmlApplicationContext，ClassPath指的是classes的路径，src下的文件打包后会出现在classes文件下
        //使用UserService方式，从spring容器中获取

        /*1.加载spring-config.xml整个spring的配置文件（文件的名字自己随便取）,spring容器会自动创建对象,
         参数是文件路径，本例是源文件和配置文件在同一目录src下，通过文件名即可找到,也推荐放在src下.
        */
         ApplicationContext context =new ClassPathXmlApplicationContext("spring-config.xml");

        //这里将配置文件放入包中时对应的文件路径如下面参数所示，也就是说路径是相对classes文件的路径
        //ApplicationContext context = new ClassPathXmlApplicationContext("com/liang/spring-config.xml");

        //第二种：文件系统路径获得配置文件(绝对路径)
        //ApplicationContext context = new FileSystemXmlApplicationContext("D:\\learning\\IDEA\\Spring-test\\src\\com\\liang\\spring-config.xml");

        //第三种：使用BeanFactory，逐渐被淘汰了，用到在做了解，它采用延时加载，调用getBean()时才创建对象，前两中是加载配置文件时就创建了对象

        //2.从容器中获取userService对象,getBean()中的参数就是配置文件中的<bean>的id值
        IUserService userService1 = (IUserService) context.getBean("userService");
        IUserService userService2 = (IUserService) context.getBean("userService");
        userService1.add();
        //1和2是相同对象
        System.out.println(userService1);
        System.out.println(userService2);

    }
```

## 装配bean的三种方式
```
<!--装配bean的三种方式，装配bean就是写一个bean标签-->
    <!--第一种：new 实现类-->
    <bean id="userService" class="com.liang.service.UserServiceImpl"></bean>

    <!--第二种：通过静态工厂方法-->
    <bean id="userService2" class="com.liang.service.UserServiceFactory1" factory-method="createUserService"></bean>

    <!--第三种：通过实例工厂方法-->
    <bean id="factory2" class="com.liang.service.UserServiceFactory2"></bean>
    <bean id="userService3" factory-bean="factory2" factory-method="createUserService"></bean>
    

使用：
    @Test
    public void test2(){
        ApplicationContext context =new ClassPathXmlApplicationContext("beans.xml");
        //IUserService userService= (IUserService) context.getBean("userService2");
        IUserService userService= (IUserService) context.getBean("userService3");
        userService.add();

    }
```

## bean的作用域

1. singleton 在spring IoC容器中仅仅存在一个bean实例，bean以单例方式存在，默认值
2. prototype 每次从容器调用bean时都返回一个新的实例，即每次getBean()相当于new XxxBean()

```
<!--通过scope确定bean的作用域，默认是singleton-->
<bean id="userService" class="com.liang.service.UserServiceImpl" scope="prototype"></bean>
```

## bean的生命周期（具体用到在学习）
需要对多个对象进行相同的逻辑处理时可以利用生命周期来完成，具体用到在学习



## AOP
面向切面的编程
>取代传统的继承来提高代码的可重用性

![image](https://note.youdao.com/yws/api/personal/file/2B0DC8603D5A4E5592080E66378FBB37?method=download&shareKey=3576544ce64e2468ba43924827255d22)


两种实现方式
1. 接口+实现类：可以采用jdk的动态代理Proxy
2. 不管有没有接口:可以采用cglib字节码增强，它所需要的包spring-core已经整合了

第一种实现方式，必须要有接口，下面代码没有给出接口，但是实际运行是需要的
```

//生成代理
package com.liang.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @author liang
 * Date 2019/4/28-18:41
 */
public class MyBeanFactory {

    public static IUserService createUserService() {
        //1.目标类
        IUserService iUserService = new UserServiceImpl();
        //2.切面类
        MyAspect myAspect = new MyAspect();
        //3.代理类-将目标类和切面类结合
        /**
         * public static Object newProxyInstance
         * (ClassLoader loader,     类加载器，一般写当前加载类
         * Class<?>[] interfaces,   代理类所需要实现的接口
         * reflect.InvocationHandler h)  处理类，一般写匿名类
         *
         */
        IUserService proxyUserService= (IUserService) Proxy.newProxyInstance(MyBeanFactory.class.getClassLoader(),
                iUserService.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        myAspect.before();
                        Object obj = method.invoke(iUserService, args);
                        myAspect.after();


                        return obj;
                    }
                });

        return  proxyUserService;
    }
}

//目标类

package com.liang.service;

/**
 * @author liang
 * Date 2019/4/27-10:17
 */
public class UserServiceImpl implements IUserService {


    private String name;

    @Override
    public void add() {
        System.out.println("增加用户---");
    }

    @Override
    public boolean del(int id) {
        System.out.println("删除用户....id:"+id);
        return true;
    }

    @Override
    public void update() {
        System.out.println("更新用户....");
    }

}



//切面类
package com.liang.service;

/**
 * @author liang
 * Date 2019/4/28-18:39
 */
public class MyAspect {
    public void before(){
        System.out.println("开启事务");
    }

    public void after(){
        System.out.println("提交事务");
    }
}

//测试
/**
 * @author liang
 * Date 2019/4/27-16:01
 */
public class Test4 {
    @Test
    public void test() {
        //动态代理
        IUserService proxyUserService=MyBeanFactory.createUserService();
        proxyUserService.add();
        System.out.println(proxyUserService.del(11));
        proxyUserService.update();


    }
}

```
第二种方式在运行时创建目标类的子类，从而完成对目标类的增强

```java

package com.liang.service;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @author liang
 * Date 2019/4/28-18:41
 */
public class MyBeanFactory {

    /**
     * 使用cglib实现aop编程
     *
     * @param
     * @return com.liang.service.IUserService
     */
    public static IUserService createUserService02() {
        //1.目标类
        IUserService iUserService = new UserServiceImpl();
        //2.切面类
        MyAspect myAspect = new MyAspect();
        //3.创建增强对象
        Enhancer enhancer = new Enhancer();
        //设置父类
        enhancer.setSuperclass(iUserService.getClass());
        //设置回调（拦截）
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                /**
                 * proxy={UserServiceImpl$$EnhancerByCGLIB$$d910b681@968}
                 * proxy代理对象是UserServiceImpl的子类
                 */
                myAspect.before();
                //执行目标类方法
                Object obj=method.invoke(iUserService,args);
                //执行代理类的方法，目标类和代理类是父子关系,没有引用iUserService，相当于解耦了
                //Object obj=methodProxy.invokeSuper(proxy,args);

                myAspect.before();
                return obj;
            }
        });

        //创建代理类对象
        IUserService iUserServiceProxy= (IUserService) enhancer.create();

        return  iUserServiceProxy;

    }
}

```

## 编写代理半自动

使用配置文件

## aop全自动编程

不需要自己写代码去将切面类和目标类结合生成代理类
使用到再了解具体用法


==AOP用在事务配置和日志记录==

## AspectJ是AOP框架

建议使用这个框架来开发AOP


切入点表达式：不需要掌握太多

通知类型：前置通知-->环绕通知-->执行方法-->后置通知-->环绕通知-->异常通知-->最终通知

XML和注解执行的通知顺序有差异，不必深究，只管注解就行了


不管有没有异常，最终通知都会执行，类似try...catch...finally

==有环绕通知就没有必要写前后置通知了==
## Aspect案例基于xml

## Aspect案例基于注解
```
1.声明使用注解
xml中配置
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">



    <!--注解的位置-->
    <context:component-scan base-package="com.liang"/>

    <!--确定aop注解生效-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

</beans>




2.容器中加入目标类和切面类
@Component

//////////////
@Component
@Aspect
public class MyAspect2 {

///////////////
@Component("carService")
public class CarServiceImpl implements ICarService{
    @Override
    public void add(){
        System.out.println("执行service增加");
    }
}

3.声明切面
@Aspect

4.声明前置通知


5.声明公共切入点

package com.liang.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * @author liang
 * Date 2019/4/28-18:39
 */
@Component
@Aspect
public class MyAspect2 {

    //声明公共的切入点
    @Pointcut("execution(* com.liang.service.CarServiceImpl.*(..))")
    public void myPointcut(){

    }


    //声明前置通知，配上切入点表达式(就是用来说明要执行的目标类方法)
    //@Before("execution(* com.liang.service.CarServiceImpl.*(..))")
    @Before("myPointcut()")//利用公共切入点
    public void myBefore(JoinPoint jp){
        System.out.println("前置通知...方法名："+jp.getSignature().getName());//连接点方法名，就是目标类的方法名

    }

    /**
     * 后置通知获取service方法执行后的返回值
     * @param jp
     * @param retValue
     * @return void
     */

    @AfterReturning(pointcut = "execution(* com.liang.service.CarServiceImpl.*(..))",returning = "retValue")
    public void myAfterReturning(JoinPoint jp,Object retValue){
        System.out.println("后置通知...."+jp.getSignature().getName());
        System.out.println("返回值:"+retValue);


    }


    //环绕通知
    @Around("myPointcut()")
    public Object myAround(ProceedingJoinPoint pjp) throws Throwable{
        System.out.println("环绕通知开启事务...."+pjp.getSignature().getName());
        Object retValue=pjp.proceed();
        System.out.println("环绕通知提交事务");
        return retValue;


    }

    //异常通知
    @AfterThrowing(pointcut = "myPointcut()",throwing = "e")
    public void myAfterThrowing(JoinPoint jp,Throwable e){
        System.out.println("异常通知..."+e.getMessage());
    }

    //最终通知
    @After("myPointcut()")
    public void myAfter(JoinPoint jp){
        System.out.println("最终通知");
    }
}


测试

public class Test5 {
    @Test
    public void test() {
        ApplicationContext context=new ClassPathXmlApplicationContext("spring-aop.xml");
        ICarService iCarService= (ICarService) context.getBean("carService");
        iCarService.add();


    }
}
```
==前后置通知和环绕通知同时存在，目标类方法只执行一次==


## JdbcTemplate （了解）--MyBatis取代它

类似于DButil的作用，操作jdbc的工具类，依赖于连接池DataSource(数据源)，导数据源相关包才能用

xml文件配置数据源和载入properties文件可以学习下，用到再学



## 事务管理

事务（transaction）：一组ABCD操作，要么全部成功，要么全部不成功

ABCD可以简单理解为对数据库表的操作,单独的查询不算事务，一定是对数据库有更改的操作

msql中事务操作可以有保存点savepoint具体用到再说
```
A
B
savepoint
C
D

出现异常可以将事务回滚到保存点，再提交，则可以完成在CD异常的情况下提交AB的需求
```


案例转账，基于spring-aop来进行事务管理，实际上就是aop那张图中的切面类可以理解成事务管理类，然后将其与目标类的方法结合

可以xml配置或者注解，这里使用注解方式


```
/////db.properties
driverClass=com.mysql.jdbc.Driver
jdbcUrl=jdbc:mysql://localhost:3306/test?characterEncoding=utf8&serverTimezone=GMT
user=root
password=root

/////spring配置文件
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">


    <!--读取配置文件-->
    <context:property-placeholder location="classpath:db.properties"></context:property-placeholder>

    <!--配置C3P0数据源
    DBCP和C3P0的数据库连接的参数的属性名不同
    -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${driverClass}"></property>
        <property name="jdbcUrl" value="${jdbcUrl}"></property>
        <property name="user" value="${user}"></property>
        <property name="password" value="${password}"></property>
    </bean>

    <!--配置dao-->
    <bean id="accountDao" class="com.liang.dao.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!--配置service-->
    <bean id="accountService" class="com.liang.service.AccountServiceImpl">
        <property name="iAccountDao" ref="accountDao"></property>
    </bean>

    <!--注解事务配置-->
    <!--配置一个jdbc事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!--开启事务注解驱动-->
    <tx:annotation-driven transaction-manager="txManager"></tx:annotation-driven>
</beans>


//////dao层
package com.liang.dao;

import org.springframework.jdbc.core.support.JdbcDaoSupport;

/**
 * @author liang
 * Date 2019/5/7-9:18
 */
public class AccountDaoImpl extends JdbcDaoSupport implements IAccountDao {
    @Override
    public void out(String outer, Integer money) {
        getJdbcTemplate().update("update account set money = money - ? where username =  ?",money,outer);

    }

    @Override
    public void in(String inner, Integer money) {
        getJdbcTemplate().update("update account set money = money + ? where username =  ?",money,inner);

    }
}


//////service层

package com.liang.service;

import com.liang.dao.AccountDaoImpl;
import com.liang.dao.IAccountDao;
import org.springframework.transaction.annotation.Transactional;

/**
 * @author liang
 * Date 2019/5/7-9:33
 */

//可以添加参数进行隔离级别等相关设置,放在类上代表该类所有方法都使用事务
@Transactional
public class AccountServiceImpl implements IAccountService {
    //spring注入
    private IAccountDao iAccountDao;

    public void setiAccountDao(AccountDaoImpl iAccountDao) {
        this.iAccountDao = iAccountDao;
    }
    @Override
    public void transfer(String outer, String inner, Integer money) {
        //使用事务后，这两个操作必须同时成功执行才会对数据库修改
        iAccountDao.out(outer, money);
        //int a=10/0;
        iAccountDao.in(inner, money);
    }

}


测试
    public void test(){
        //转账测试
        ApplicationContext context=new ClassPathXmlApplicationContext("spring-aop2.xml");
        IAccountService iAccountService= (IAccountService) context.getBean("accountService");
        iAccountService.transfer("jack","rose",1000);
    }
```
