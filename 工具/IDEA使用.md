> 当IDEA配置出错要还原时，直接删掉system和config文件，重启程序即可

>创建包时，当首字母有大写时会自动创建类，例如com.liang.Customer会在com.liang包下生成Customer类

>调用其他模块的类时，会提示错误，添加依赖即可，.iml文件可看到依赖信息，删除依赖信息会再次报错。

**快捷键**


快捷键 | 作用
---|---
Shift+Enter| 换行
Ctrl+Y| 删除
Ctrl+D|复制当前行或者选中的内容
Ctrl+Shift+F10|编译运行当前选定文件
Ctrl+alt+空格|代码提示补全（当光标移动到其他地方再移动回来时）
Ctrl+Shift+上（下）|向上（下）移动声明
Alt+Shift+上|向上移动行
Ctrl+Alt+Enter|向上开启一行
Shift+Enter|向下开启一行
Ctrl+B（或鼠标左键）|查询选中源码
双击Shift|输入查询源码/查找文件
Ctrl+Alt+v |补全返回值
Alt+左（右）|切换编辑页面
Ctrl+H|查看继承关系
Ctrl+Alt+L|格式化代码
Ctrl+P|方法参数类型提示
Ctrl+Shift+z|反撤销
Shift+上（下，左，右）|按方向选中文本
Alt+7|查看类的结构
Shift+F6|重构：修改变量名和方法名，形参名
Ctrl+Shift+U|大小写转换
Alt+Insert|生成构造器/get/set/toSting()
Alt+Enter|表明意图行动，快速解决错误
Ctrl+Q|查看文档说明
Ctrl+Shift+减号（加号）|方法收缩和展开
Alt+Z(自定义设置)|打开代码所在硬盘位置
Ctrl+Alt+T|生成if else/try catch等代码
Ctrl+Alt+F|局部变量抽出成为成员变量
Ctrl+Shift+F|查找（全局）
Ctrl+F|查找当前文件
Ctrl+Alt+Shift+U|查看类的继承结构图
Ctrl+Shift+H|查看方法的多层重写结构
Ctrl+Alt+M|抽取方法
Ctrl+E|打开最近修改的文件
Ctrl+F4|关闭当前代码栏
Ctrl+Shift+F4|关闭所有代码栏
F2|定位错误位置
Ctrl+Shift+v|显示最近复制的内容，并且选择其中一个粘贴
Ctrl+Alt+H|查看方法在哪里被调用




**代码模板**

IDEA中模板所处位置setting -->editor -->live template/postfix completion(不可修改)

常用模板如下表格：

代码|内容
---|---
psvm|main
sout|println
soutp|打印函数的参数(做测试用)
soutm|打印"当前类名.当前方法名"
soutv|打印本代码上方最近的变量名和值
i.sout|打印System.out.println(i)
fori|遍历代码（Enter跳跃光标）
itar/iter|增强for循环（无法获取索引的循环，适合集合,只需要遍历时）
list.for|生成集合List的for循环(增强)
list.fori/list.forr（倒序）|普通for循环
ifn/i.null|if (i == null)
inn/i.nn|if (i != null)
prsf|private static final
psf|public static final
psfi|public static final int
psfs|public static final String

>live templates 主要关注iterations,output,other,plain相关
postfix completion关注java


**插件推荐**

GsonFormat
主要JSON字符串实例化成类

使用：
自定义个javaBean(无任何内容，就一个空的类)
复制你要解析的json
然后alt+insert


**注意事项**

setting中有些设置只针对当前project，看设置界面上方条目有提示。在default setting中可进行全局设置。完成设置后，新建项目会默认会根据default setting中的设置进行设置


**其他配置参看课件**


### 使用IDEA的相关问题

1. 利用maven创建web项目时，java,test等文件需要自己添加
2. 创建spring配置文件时，不需要手动添加命名空间，写对应标签时，会自动添加所需要的命名空间
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package=""
</beans>

自动添加xmlns:context
```









