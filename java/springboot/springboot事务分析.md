## 参考文档

[事务传递性](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486668&idx=2&sn=0381e8c836442f46bdc5367170234abb&chksm=cea24307f9d5ca11c96943b3ccfa1fc70dc97dd87d9c540388581f8fe6d805ff548dff5f6b5b&token=1776990505&lang=zh_CN#rd)

[spring事务总结](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/spring-transaction)

[使用Seata彻底解决Spring Cloud中的分布式事务问题](https://macrozheng.github.io/mall-learning/#/cloud/seata)

## 问题现象

1、单体项目多个数据源，开启事务的方法中调用了不同数据库的dao层，如何解决事务回滚问题？

2、分布式事务又是如何解决？