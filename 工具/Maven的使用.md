[TOC]

## 跨平台项目管理工具

>管理jar包

>仓库用于存放从网上下载的jar包，仓库默认路径在用户目录下C...\\.m2\repository,默认从中央仓库下载包。<br>
mvn help:system下载一些默认包


## maven项目结构


```
ProjectName
|-src<br>
|   |-main<br>
|   |   |-java————存放项目的.java文件<br>
|   |   |-resources————存放项目资源文件,如spring配置文件<br>
|   |-test<br>
|       |-java————存放所有测试.java文件，如JUnit测试类<br>
|       |-resources————测试资源文件<br>
|-target————==目标文件输出位置==例如.class,.jar,.war文件<br>
|-pom.xml————maven项目核心配置文件<br>
```

## maven常用命令

1. mvn compile,完成编译，生成文件在target下
2. mvn clean,删除target目录
3. mvn test做单元测试
4. mvn package,完成打包操作，在target中生成jar或war文件
5. mvn install,将打包好的jar包安装到本地仓库
6. 组合命令：mvn clean compile/test/package/install


## GAV确定唯一jar包

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.0.7.RELEASE</version>
</dependency>
```

## 依赖
A 依赖 B，A默认可以继承B的依赖,即<scope>compile</scope>，如果B项目的某一依赖添加参数<scope>test</scope>,则A无法继承该依赖。参数provided代表不要打包到jar包

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.0.7.RELEASE</version>
    <!-- 是否向下传递依赖，true不传递，false传递（默认），类似上述参数-->
    <optional>true/false</optional>
    <!-- 主动排除一些继承的依赖-->
    <exclusions>
        <exclusion>
            <groupId></groupId>
            <artifactId></artifactId>
        </exclusion>
    </exclusions>
</dependency>

```
有依赖关系的项目打包时，被依赖的项目也事先要打包

可以通过配置pom.xml插件来改变项目编译的jdk版本


## 继承，提取相同的配置

创建父工程必须使用pom打包方式

创建子工程时，添加父工程的配置即可使用父工程的依赖

父工程一般不写代码，只用于统一管理依赖和版本号

统一管理版本号的方法
```xml
    <properties>

        <junit.version>4.12</junit.version>
        <!--....可继续添加多个-->
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/junit/junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <!--<version>4.12</version>-->
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```


父工程的pom.xml

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.20</version>
        </dependency>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```


module1的pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
    </dependency>
</dependencies>
```

其他模块类同module1。这样一来，子模块无需引入版本号就能够实现依赖，当我们需要修改某个jar的版本时，只需要在父工程中修改即可。

==父工程没有标签`<dependencyManagement>`时，子工程会自动继承父工程的所有依赖，添加该标签后，子工程想要使用就必须显示声明该依赖，不需要写版本号==



## 创建聚合项目

先创建父工程。再在父工程下创建maven module

打包直接打包父工程，它会自动按父工程的pom.xml中的模块顺序打包所有模块


## maven默认编译使用的jdk相关问题

maven编译时会使用插件，该插件在pom中不指定jdk版本时，默认是jdk1.5（也可能他是其它版本）。所以一般需要在配置文件中指定jdk版本

1. 全局修改编译版本


setting.xml中增加如下代码
```
    <profiles>
    <profile>
      <id>jdk-1.8</id>
      <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
      </activation>
      <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
      </properties>

    </profile>
    </profiles>
```

## 一些问题

maven中的test，具体开发如何去用（是配合JUnit吗）和普通的对当前文件编译输出有何不同？<br>
可能是不一定每个java文件都去写控制台输出语句，这样测试不方便<br>
模块之间的依赖和maven项目之间的依赖配置，对打包项目的影响



## 配置出错

```
报错：expected START_TAG or END_TAG not TEXT

配置文件中的代码，我直接从博客上复制过来的导致出错，应该是格式出错，重新手动输入一遍即可
```
## maven打包方式

通过packaging标签可以更改war,jar,pom

```
  <groupId>com.wuhan</groupId>
  <artifactId>MyJspProject</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
```


## maven的提示错误
```
Failed to create a Maven project ‘…pom.xml’already exists in VFS
```
解决方法，清除缓存即可

## 下载源码
IDEA中先File--setting--import勾选自动下载源码，让后对maven项目reimport即可对已经导入的包下载源码