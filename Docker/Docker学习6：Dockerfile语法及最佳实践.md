# Docker学习6：Dockerfile语法及最佳实践

## FROM：

docker搭建的基础

```dockerfile
FROM scratch  #建立的是base image
FROM centos   #使用的base image
FROM ubuntu:14.04
```

## LABEL：

docker image的Metadata

```dockerfile
LABEL maintainer="haohao"
LABEL version="1.0"
LABEL description="This is description"
```

## RUN：

在基础上运行的命令

```dockerfile
RUN yum update && yum install -y vim \
    python-dev   #反斜线换行
```

```dockerfile
RUN apt-get update && apt-get install -y perl \
    pwgen --no-install-recommends && rm -rf \
    /var/lib/apt/lists/*  #清理cache
```

## WORKDIR：

设定当前工作目录(在哪个文件夹执行RUN的命令)

```dockerfile
WORKDIR /root
```

> 没有这个目录的话就会新建

## ADD：

> 将指定的文件（当前系统中的文件）移入到指定文件夹（docker中的文件夹）
>
> `ADD 目标文件 目标目录`

```dockerfile
ADD hello /
```

```dockerfile
ADD test.tar.gz /    #添加到根目录并解压
```

```dockerfile
WORKDIR /root
ADD hello test/    # /root/test/hello
```

```dockerfile
WORKDIR /root
COPY hello test/
```

<font color=#FF0033>大部分情况，COPY优于ADD。ADD除了COPY还有额外功能（解压）。添加远程文件、目录请使用curl或者wget</font>

## ENV：

> 设置常量

```dockerfile
ENV MYSQL_VERSION 5.6    #设置常量
RUN apt-get install -y mysql-server= "${MYSQL_VERSION}" \
    && rm -rf /var/lib/apt/lists/*    #引用常量 
```

## VOLUME&EXPOSE

> 存储和网络