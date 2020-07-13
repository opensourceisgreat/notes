### docker容器中输入中文

#### 1、问题描述

无法在容器中输入中文

#### 2、问题原因

容器中的字符集如下：

```shell
root@8d5503532139:/# locale
LANG=
LANGUAGE=
LC_CTYPE="POSIX"
LC_NUMERIC="POSIX"
LC_TIME="POSIX"
LC_COLLATE="POSIX"
LC_MONETARY="POSIX"
LC_MESSAGES="POSIX"
LC_PAPER="POSIX"
LC_NAME="POSIX"
LC_ADDRESS="POSIX"
LC_TELEPHONE="POSIX"
LC_MEASUREMENT="POSIX"
LC_IDENTIFICATION="POSIX"
LC_ALL=

```

其中`LANG`没有设置正确，或者说生成容器时，就没有管这个字符集设置，因此导致无法输入中文

#### 3、解决方案：

1. 容器已经创建并启动，输入下面的命令即可

```shell
docker exec -it 8d5503532139  env LANG=C.UTF-8 /bin/bash
```

2. 在创建自己的docker镜像时加入如下语句

   在`Dockerfile`中加入：

   ```
   ENV LANG=C.UTF-8
   ```

设置后的结果

```shell
root@8d5503532139:/# locale
LANG=C.UTF-8
LANGUAGE=
LC_CTYPE="C.UTF-8"
LC_NUMERIC="C.UTF-8"
LC_TIME="C.UTF-8"
LC_COLLATE="C.UTF-8"
LC_MONETARY="C.UTF-8"
LC_MESSAGES="C.UTF-8"
LC_PAPER="C.UTF-8"
LC_NAME="C.UTF-8"
LC_ADDRESS="C.UTF-8"
LC_TELEPHONE="C.UTF-8"
LC_MEASUREMENT="C.UTF-8"
LC_IDENTIFICATION="C.UTF-8"
LC_ALL=

```

