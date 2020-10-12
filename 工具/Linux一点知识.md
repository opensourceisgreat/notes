### Linux 使用技巧

1、strace 跟踪系统调用

```shell
-ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的filename.pid中,pid是各进程的进程号. 
strace -ff -o out java SocketNIO
#java SocketNIO 运行JAVA文件,注意java8运行的文件，线程输出文件中第二个才是main线程的系统调用信息
```

2、man 2，man 3

man 2 可以查看系统调用open,write 说明

man 3 是一些库函数说明

3、set nu 显示vi打开后的行数

/匹配，可以匹配到对应内容的位置