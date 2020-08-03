# 一、docker 打包

本文参考的是该项目地址：[https://macrozheng.github.io/mall-learning/#/](https://macrozheng.github.io/mall-learning/#/)

docker官网地址：[https://docs.docker.com/reference/](https://docs.docker.com/reference/) ，可以查看Dockfile相关命令设置，以及docker-compose的使用

该项目已经详细介绍了打包过程，我在这里就只阐述一些注意点：

1、打包springboot项目，其中项目需要输出日志，Dockfile中的注意点：

==特别注意ENTRYPOINT启动参数，输出日志文件时保证时间是正确的时区==

```dockerfile
# 该镜像需要依赖的基础镜像
FROM java:8
# 将当前目录下的jar包复制到docker容器的/目录下
ADD xueyuan-service-0.0.1-SNAPSHOT.jar /xueyuan-service-file.jar
# 运行过程中创建一个xueyuan-service-file.jar文件
RUN bash -c 'touch /xueyuan-service-file.jar'
# 声明服务运行在8080端口
EXPOSE 8080
# 指定docker容器启动时运行jar包,-Duser.timezone=GMT+8设置JVM时区,输出日志文件时保证时间是正确的时区
ENTRYPOINT ["java", "-jar","-Duser.timezone=GMT+8","/xueyuan-service-file.jar"]
# 指定维护者的名字
MAINTAINER liang
```

如果不设置-Duser.timezone=GMT+8，改为创建容器时添加参数，`docker run`命令中的`-e TZ="Asia/Shanghai"`。这种情况下，日志打印的时间是否正确？`-e 设置环境变量，在linux中设置TZ会让系统优先使用该时区`

这里就有问题了，`-v /etc/localtime:/etc/localtime`和`-e TZ="Asia/Shanghai"`需要一起用吗，两者到底有什么区别

事实上，都是为了确保宿主机和容器时区能同步上。只不过 JVM 读取 Linux 上的时区时，按顺序从这里面取，取到就作为 JVM 时区

2、使用docker-compose.yml，启动一系列服务注意点：

`- /etc/localtime:/etc/localtime`最好是带上，保证容器时区与宿主一致，

`command:...`参数可以覆盖Dockerfile中的ENTRYPOINT命令（即通过Dockerfile生成的镜像，在启动时会默认执行的命令）

`services`下的`mysql`和`redis`两个服务是在同一个默认网络（运行docker-compose.yml时会生成一个默认的网络，而网络名称就是docker-compose.yml所在目录的名称，例如目录名称为root,则网络名称root_default）中的，两个容器就可以在一个局域网络中在redis容器中`ping mysql(服务名称)`是可以直接ping通的。

```yaml
version: '3' #docker-compose版本，参考官网
services:
  mysql: #服务名称（自定义）
    image: mysql:5.7.28
    container_name: mysql
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456 #设置root帐号密码
    ports:
      - 3306:3306
    volumes:
      - /etc/localtime:/etc/localtime #同步时区
      - /mydata/mysql/data/db:/var/lib/mysql #数据文件挂载
      - /mydata/mysql/data/conf:/etc/mysql/conf.d #配置文件挂载
      - /mydata/mysql/log:/var/log/mysql #日志文件挂载
  redis:
    image: redis
    container_name: redis
    command: redis-server --appendonly yes
    volumes:
      - /etc/localtime:/etc/localtime
      - /mydata/redis/data:/data #数据文件挂载
    ports:
      - 6379:6379
```

3、docker中cmd的一些注意点：

启动eureka容器服务，通过`--net deploy_default`放在和`xueyuan-service`服务一个网络中。

原因是`xueyuan-service`服务需要找到注册中心服务(eureka)，就要保证两个服务在一个网络中。

启动`xueyuan-service`服务时，由于需要使用mysql服务，使用参数`--link mysql:db`，那么`ping db`就可以通了，注意只有在本容器中才能通过`ping db`才能成功，可以理解为本容器针对mysql服务取了个别名，设置了域名映射。当然如果不设置`--net deploy_default`就无法让`xueyuan-service`容器和mysql容器在一个网络中（注意这里mysql和redis服务通过`docker-compose`启动的，在`deploy_default`网络中），也就无法ping通，`--link`参数也就没啥用了。

==在使用docker-compose.yml进行启动时，如果有多个文件（多个swarm），最好放在一个目录下，这样就在同一个网络中了==

多个容器只要在同一个网络中就可以直接ping容器名，也可以ping通；不需要`--link`



```bash
docker run -p 8080:8080 \
--name xueyuan-service \
--link mysql:db \
--link redis:redis \
--link xueyuan-eureka:eureka \
--net deploy_default \
-v /etc/localtime:/etc/localtime \
-v /mydata/xueyuan-service/log:/var/log/whu_log \
-d xueyuan-service-file:0.0.1-SNAPSHOT


docker run -p 8003:8003 \
--net deploy_default \
--name eureka \
-v /etc/localtime:/etc/localtime \
-d xueyuan-eureka-file:0.0.1-SNAPSHOT
```



docker-compose.yml中的配置`external_links`类似CMD中的`--link`

```yaml
version: '3'
services:
  mall-admin:
    image: mall/mall-admin:1.0-SNAPSHOT
    container_name: mall-admin
    ports:
      - 8080:8080
    external_links: 
      - redis:redis #可以用redis这个域名访问redis服务    
      - mysql:db #可以用db这个域名访问mysql服务
```

# 二、容器时区，宿主时区和JVM时区

### 1、参考文档

[JVM读取时区](https://www.cnblogs.com/KJXY/articles/12737488.html)

[docker容器 java 默认读取系统时区问题](https://blog.csdn.net/SoberChina/article/details/84849860)

### 2、问题现象

springboot 输出日志文件时，日期描述使用的不是 GMT+8 时区

### 3、问题原因

JVM获取时区在Linux系统上，大概过程为以下几步：

1. 先找`TZ`变量，没有，到2。
2. 读`/etc/timezone`，没有到3。
3. 比较`/etc/localtime`文件与"`/usr/share/zoneinfo`目录下所有时区文件，如果有一致的，就为该时区，如果没有，到4,
4. 默认为标准GMT。

### 4、解决方案

>第一种: 挂载时区文件；
>
>第二种: 配置`TZ`环境变量；`docker run`命令中的`-e TZ="Asia/Shanghai"`
>
>第三种: Dockerfile 添加`RUN echo "Asia/Shanghai" > /etc/timezone` 配置
>
>第四种：jvm 参数 -Duser.timezone=GMT+8

# 三、jvm系统变量，操作系统变量

**参考文档**

[SpringBoot系列——利用系统环境变量与配置文件的分支选择实现“智能部署”](https://www.cnblogs.com/huanzi-qch/p/10411581.html)



1、java -Dfoo=xx

java 程序可以通过 `System.getProperty("foo")`获取系统属性foo的值，这里的值为 xx

-D作用如下

> Set a system property value. If value is a string that contains spaces, you must enclose the string in double quotes:

2、springboot启动时，读取占位符的变量，可以读取操作系统的环境变量，本文件没有对应的名字就去操作系统找

`docker run 中-e 'spring.profiles.active'=prod`，先在 docker 容器中设置环境变量 spring.profiles.active ，然后对应的yaml文件如下

```yaml
spring:
  profiles:
    active: ${spring.profiles.active:dev} #选择配置分支，先读取操作系统环境变量，如果没有则默认值为 dev
```

JAVA程序`System.getEnv()` 可以获取操作系统的环境变量