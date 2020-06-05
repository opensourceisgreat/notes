


## Tomcat
start.bat启动

>端口号默认8080(容易被占用)，可在conf/server.xml文件下更改

```xml
    <!-- A "Connector" represents an endpoint by which requests are received
         and responses are returned. Documentation at :
         Java HTTP Connector: /docs/config/http.html
         Java AJP  Connector: /docs/config/ajp.html
         APR (HTTP/AJP) Connector: /docs/apr.html
         Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
    -->
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

>常见状态码：404（资源不能存在）,500（代码写错）<br>

项目在Tomcat发布的文件结构
```xml
webapps
|-projectName
|   |-WEB-INF
|      |-lib
|      |-classes
|      |-web.xml

web.xml文件中可以指定项目默认启动的网页
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.xhtml</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>

```


修改server.xml文件,将项目配置到webapps以外的目录
```xml
 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        <!-- docBase是项目的实际路径，path是虚拟路径（绝对路径，相对路径（相对于D:\tools\apache-tomcat-8.5.40\webapps）），一般就写/projectname-->
        <Context docBase="" path=""    />
        
</Host>
```

conf\Catalina\localhost中新建“项目名.xml”中新增<Context/>，如果改名为ROOT.xml,路径可以不要项目名，直接对项目下的jsp访问


配置虚拟主机

修改conf\server.xml
```xml
<Engine name="Catalina" defaultHost="www.test.com">
      <Host name="www.test.com"  appBase="项目文件的实际路径">
       <Context docBase="项目文件的实际路径" path="/"    /> 
      </Host>
```
这里path可以用/是因为appBase已经定位在项目文件路径了，path是相对于这个路径配置，这样改虚拟路径和上面的例子就是一个意思了

C:\Windows\System32\drivers\etc\hosts，增加127.0.0.1  www.test.com

访问时注意添加端口号，可以理解为localhost变为www.test.com，如果是80端口就不需要了

流程：
www.test.com -> host找映射关系-->server.xml找Engine的defaultHost->通过"/"映射到项目文件的实际路径。所以访问www.test.com:8080/即可访问到对应项目。path如果是/projectname，访问地址就是www.test.com:8080/projectname

>第一次访问文件会比较慢，服务器要编译成class文件，第二次就比较快，修改文件时会重新编译


>WEB-INF文件下的内容无法直接通过浏览器访问，只能通过内部的请求转发访问，重定向是不行的

## 字符编码
jsp文件的编码（文件中的pageEncoding属性）

浏览器读取jsp文件的编码，jsp中的<content>属性

文本编码，文件保存的编码IDE统一设置成UTF-8即可

## 交互问题
get提交方式，form默认的，或者指定method="get",超链接,地址栏提交方式也是get,?uname=..&upassword=...

数据有限，不安全

post,上传文件一般用post

tomcat7可能出现的问题，8会默认utf-8
>get方式请求出现编码问题，server.xml进行修改<br>
post方式,request.setCharacterEncoding("UTF-8");<br>
注意，响应时response.setCharacterEncoding必须在输出对象产生之前


Cookie,服务端生成，再发送给客户端保存，类似本地缓存。<br>
respose.addCookie<br>
转发或者重定向都可以给客户端<br>
客户端获取request.getCookies();必须拿到所有cookie

## Mysql

>java中调用存储过程和存储函数用到的时候再说
>
>存储大型数据，可以存储路径，或者用text类型存储,实际用到再学习使用

>preparedStatement可以防止sql注入

## JavaBean

两种用途
1. 封装业务逻辑
2. 封装数据
3. 好用的包lombok,用注解生成get,set,有参或者无参构造函数，不需要自己写了，POJO很适合用，源码没有get,set编译后字节码文件会有get,set,所有scope=provided没问题，不需要在打包是将该jar包打入到项目中，便已打包后的项目都是字节码文件。但是有一个问题，案例中的POJO序列化了，强调必须序列化？没懂

在分布式系统中，此时需要把对象在网络上传输，就得把对象数据转换为二进制形式，需要共享的数据的 JavaBean 对象，都得做序列化

## Servlet

1. Servlet2.5需要配置web.xml
2. Servlet3.0在类前加注解@WebServlet(value = "/LoginServlet"),IDEA中必须这么写

2.5需要进行下列配置
```xml
  <servlet>
    <servlet-name>XXX</servlet-name>
    <servlet-class>com.liang.LoginServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>XXX</servlet-name>
    <url-pattern>/LoginServlet</url-pattern>
  </servlet-mapping>
```
项目的根目录：webapp,main/java处于同一级别,source root,IDEA中可以设置文件夹具体是什么类型
```
<a href="LoginServlet">所在的jsp文件是在webapp目录中，因此发出的请求是去请求项目的根目录.

Servlet流程：
请求--> <url-pattern>/LoginServlet</url-pattern> -->根据<servlet-mapping>中的<servlet-name>XXX</servlet-name>去和<servlet>中<servlet-name>XXX</servlet-name>匹配，然后寻找到<servlet-class>，将请求交给它执行
```

区分：
1. 上述web.xml中的开头/,代表项目根路径:http://localhost:8080/MyJspProject/，注解中的/也同理
2. jsp中的开头/，代表服务器根路径:http://localhost:8080/,<a href="/...">，所以jsp文件中的跳转不用/开头

>B站有一个源码分析，需要学习时看。[servlet源码分析](https://www.bilibili.com/video/av29086718/?p=22)

## jsp内置对象在servlet中获取

```java
out: PrintWriter out=response.getWriter();
session: request.getSession();
application: request.getServletContext();
```

## 更改tomcat项目默认启动的首页

修改web.xml,将servlet写入即可，相当于先启动servlet
```xml
<web-app>
  <welcome-file-list>
    <welcome-file>AddStudentServlet</welcome-file>
  </welcome-file-list>
</web-app>
```

## 三层架构优化

1. 加入接口，面向接口开发---service,dao加入接口
2. DBUtil,新建util包放一些帮助类，简化dao层

写法:接口 x = new 实现类()

多个方法相同代码多进行提炼


## 过滤器（拦截器，原理相同）

>让一个普通类具有特定功能必须通过继承，实现接口或者注解

```xml
配置过滤器可以注解或者web.xml

  <filter>
    <filter-name></filter-name>
    <filter-class>MyFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name></filter-name>
    <!-- /*拦截所有的请求，/myServlet只拦截访问myServlet的请求，一般开发这种就所有请求都过滤写/*-->
      <url-pattern>/*</url-pattern>
      <!--还可以指定拦截什么类型的请求-->
  </filter-mapping>

多个过滤器形成的过滤器链根据<filter-mapping>的位置决定谁先拦截
    custom-->filter1-->filter2-->server,request
    custom<--filter1<--filter2<--server,response
```





```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--全局配置，当你只需要一个配置就够了的情况，没有多个dispatcherServlet
    -->
    <!--<context-param>-->
        <!--<param-name>contextConfigLocation</param-name>-->
        <!--<param-value>/WEB-INF/applicationContext.xml</param-value>-->
    <!--</context-param>-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--配置文件的位置和名称,如果不配置，系统自己在WEB-INF下找[servlet-name]-servlet.xml文件，如dispatcher-servlet.xml-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!--web应用被加载时创建，而不是等到servlet请求时-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <!--/*的作用所有请求都来这，包括.jsp文件的请求也会来这，导致去执行DispatcherServlet的相关方法，进而去找该请求对应的控制器
            /的作用就是排除了.jsp文件
        -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

The `/*` on a servlet overrides all other servlets, including all servlets provided by the servletcontainer such as the default servlet and the JSP servlet. Whatever request you fire, it will end up in that servlet.

过滤器中的/和/*同理

[/和/*的解析参考](https://stackoverflow.com/questions/4140448/difference-between-and-in-servlet-mapping-url-pattern)



[拦截器和过滤器的区别](<https://blog.csdn.net/zxd1435513775/article/details/80556034>)

## 待日后学习

上述视频p31往后的上传，下载，分页，aj，监听，拦截等内容，用到的时候回头补充