# 1.MyBatis的框架核心

1. 全局配置文件和映射文件

2. [官方文档](http://www.mybatis.org/mybatis-3/index.html)

   

# 2.简单试用MyBatis

环境准备：导包

日志使用的是log4j

1.全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jc.jdbc.Driver"/>
                <!--&amp;就是&-->
                <property name="url" value="jdbc:mysql://localhost:3306/test?characterEncoding=utf8&amp;serverTimezone=GMT"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    </mappers>
</configuration>
```



## 2.1参数

![参数](https://note.youdao.com/yws/api/personal/file/3D84FA6BE29B46EF94C2C14C164F5A02?method=download&shareKey=85f60148afd88c174e0b13269c9a9a02)

1. order by 时使用$，其他地方尽量不用，==#会比$多传入引号==

2. 如果 parameterType 只有一个简单参数，默认情况下，#{ }中可以写任意的名字；${ }中只能用value来接收，模糊查询可以用$，这样就不必考虑引号的麻烦

3. mapper接口中的方法参数可以添加注解@Param("参数别名")，xml映射文件可以通过${参数别名},#{参数别名}来接收参数，此时不需要指定映射文件中的参数类型，==可以用于获取多参数，推荐使用该方法==

4. 还可以使用map封装多个参数，xml直接指定参数类型为hashmap，${key}或#{key}就可以获取参数值

5. ```xml
   #{property,javaType=int,jdbcType=NUMERIC}
   这里就是对#{}代表的参数进行详细说明，一般不用，在属性是hashmap这种复杂类型，显示指定javaType=hashmap来告诉mybatis进行正确的类型处理(TypeHandler)，很少遇到这种情况
   ```



resultType/resultMap

更新数据库信息会自动返回受影响的行数，不需要resultType，mapper接口定义int返回类型就行

查询语句返回单个或多个POJO（普通的java对象），resultType="类名"，只不过mapper接口的返回类型分别设置为POJO和List

查询语句返回简单类型数据(就一个值)，如select count(*).....，resultType="int"

使用resultType,POJO的属性名和返回的列名一致，POJO的属性名才会有值，不然没法映射，你可以选择不映射某一列，但是映射就必须保持一致，如果全都不一致，返回数据后，并不会创建POJO对象，mapper接口返回null，但是只要有一个能够映射成功,会有对象返回，但是属性值只有它有值，其它属性都没有值

resultMap解决查询出来的列名和POJO属性名不一致时，仍要完成映射的问题（比如sql语句经常会取别名）

```xml
<!--第一种解决方案，将得到的列名取别名，使其与POJO属性名一致（实际上就是要mybatis好调用set方法）-->
<select id="selectUsers" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>

<!--第二种：通过resultMap解决-->
<resultMap id="userResultMap" type="User">
  <!--这个id标签，没啥特别，表明这是表的主键，也可以用result-->
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```







## 2.2案例

mapper包取代dao,下面是参考目录结构

```
  /lib                <-- MyBatis *.jar 文件在这里。
  /src
    /org/myapp/
      /action
      /mapper           <-- MyBatis 配置文件在这里, 包括映射器类, XML 配置, XML 映射文件。
        /mybatis-config.xml
        /BlogMapper.java
        /BlogMapper.xml
      /model
      /service
      /view
    /properties       <-- 在你 XML 中配置的属性文件在这里。
```



案例：

目录结构：
```
-src
--main
----java
-------com.liang
-------------mapper（就是dao做的事）
---------------UserMapper
---------------UserMapper.xml
--------------model
----------------User(就是普通的javabean)
-------db.properties
-------mybatis-config.xml
```

UserMapper:

```java
package com.liang.mapper;

import com.liang.model.User;

/**
 * @author liang
 * Date 2019/5/8-19:39
 */
public interface UserMapper {
    /**
     * 保存用户
     * @param user
     * @return int 返回收影响的行数
     */
    public int save(User user);
    /**
     * 根据id查找用户
     * @param id
     * @return com.liang.model.User
     */
    public User findUserById(int id);
    /**
     * 根据姓名查找用户
     * @param username
     * @return java.util.List<com.liang.model.User>
     */
    public List<User> findUserByName(String username);

}
```

UserMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.liang.mapper.UserMapper">
    <!--  参数是复杂类型时，#{username}中的username指定的是User对象的属性值username
    这样理解，#{}就是用来绑定参数的属性值，只不过简单类型没有属性，直接绑定参数值

    useGeneratedKeys="true"
    仅对 insert 和 update 有用，这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键
    （比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。

    keyProperty="uid"，仅对 insert 和 update 有用，设置对象属性值（这里是属性uid，写错是无法赋值的）
    这两个参数的应用前提是数据库支持自动生成值。

    当然也可以用<selectKey>元素完成上述功能，只不过需要自己写sql获取表中的自增长的属性值，具体参见官方文档

    个人认为只考虑自增长属性赋值时第一种简单，第二种是在你有别的需求时可以考虑使用

	单个参数，parameterType="com.liang.model.User"可以省略
    -->
    <insert id="save" parameterType="com.liang.model.User" useGeneratedKeys="true" keyProperty="uid">
      insert into bs_user(username,birthday,sex,address) values (#{username},#{birthday},#{sex},#{address})
    </insert>
    <!--参数是简单类型时#{uid}就当做是一个?,uid没啥意义-->
    <select id="findUserById" parameterType="int" resultType="com.liang.model.User">
      select * from bs_user where uid = #{uid}
    </select>
    
    <!--返回值类型还是填写user,而不是list，返回的每一条数据都需要封装在user中，查询返回多条数据时，自动将多个数据封装在list集合中-->
    <select id="findUserByName" parameterType="string" resultType="com.liang.model.User">
      select * from bs_user where username = #{username}
    </select>

</mapper>
```

mybatis-config.xml：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--引入外部配置文件-->
    <properties resource="db.properties"></properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <!--&amp;就是&-->
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 使用相对于类路径的资源引用 -->
    <mappers>
        <mapper resource="com/liang/mapper/UserMapper.xml"/>
    </mappers>
    <!-- 使用类名,需要有对应文件名的xml文件，如果没有xml文件，可以在UserMapper的方法上添加注解，但是不推荐这样做因为sql语句复杂后，使用注解不灵活-->
    <!--<mappers>-->
    	<!--<mapper class="com.liang.mapper.UserMapper"/>-->
    <!--</mappers>-->

    <!--&lt;!&ndash; 将包内的映射器接口实现全部注册为映射器 &ndash;&gt;-->
    <!--<mappers>-->
        <!--<package name="com.liang.mapper"/>-->
    <!--</mappers>-->
</configuration>
```

db.properties:

```properties
#mysql6以上的驱动
driver=com.mysql.cj.jdbc.Driver
# &amp;必须还原成&，仅仅在xml中需要这样写
url=jdbc:mysql://localhost:3306/test?characterEncoding=utf8&serverTimezone=GMT
username=root
password=root
```



测试

```java
public class AppTest {
    SqlSession session;

    @Before//测试执行前
    public void before() throws IOException {
        //从 XML 中构建 SqlSessionFactory
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        //从 SqlSessionFactory 中获取 SqlSession
        session = sqlSessionFactory.openSession();

    }

    @After//测试执行后
    public void after() {

        session.close();
    }

    /**
     * Rigorous Test :-)
     */
    @Test
    public void shouldAnswerWithTrue() {
        UserMapper um = session.getMapper(UserMapper.class);
        User user = new User("xiaokkkk", "女", new Date(), "武汉");
        um.save(user);
        session.commit();
        
        //List<User> al = um.findUserByName("xiaokkkk");
        //System.out.println(al);
    }
}
```



通过配置日志可以再控制台打印具体sql语句的执行情况,这里就不配置了,springboot中再使用springboot的配置文件来做这个工作

==insert，update，delete会返回受影响的行数，为1时，可以返回布尔true,但是批量操作>1时为false==





# 3.类型别名（typeAliases）

类型别名是为 Java 类型设置一个短的名字。 它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。例如：==用在全局配置文件中==

```xml
<configuration>
    <typeAliases>
      <typeAlias alias="Author" type="domain.blog.Author"/>
      <typeAlias alias="Blog" type="domain.blog.Blog"/>
      <typeAlias alias="Comment" type="domain.blog.Comment"/>
      <typeAlias alias="Post" type="domain.blog.Post"/>
      <typeAlias alias="Section" type="domain.blog.Section"/>
      <typeAlias alias="Tag" type="domain.blog.Tag"/>
    </typeAliases>
</configuration>
```

当这样配置时，`Blog` 可以用在任何使用 `domain.blog.Blog` 的地方。

这是一些为常见的 Java 类型内建的相应的类型别名。它们都是不区分大小写的，注意对基本类型名称重复采取的特殊命名风格。

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

每一个在包 `domain.blog` 中的 Java Bean，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 `domain.blog.Author` 的别名为 `author`；



==配置之后，映射文件中参数类型和返回值类型都可用别名了==



# 4.学到动态sql了以后用到在继续学

[视频内容](<https://www.bilibili.com/video/av49045938/?p=15>)

# 5.逆向工程

   mybatis需要程序员自己编写sql语句，mybatis官方提供逆向工程，可以针对单表自动生成mybatis执行所需要的代码（mapper.java、mapper.xml、pojo…），可以让程序员将更多的精力放在繁杂的业务逻辑上。

        企业实际开发中，常用的逆向工程方式：由数据库的表生成java代码。
    
        之所以强调单表两个字，是因为Mybatis逆向工程生成的Mapper所进行的操作都是针对单表的，也许你可能会觉得那这就有点鸡肋了，但是在大型项目中，很少有复杂的多表关联查询，所以作用还是很大的。

