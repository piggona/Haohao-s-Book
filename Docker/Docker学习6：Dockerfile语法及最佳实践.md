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

在容器(FROM)基础上运行的命令,并建立一个新的image

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

## ADD&COPY：

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

## CMD&ENTRYPOINT

> 两者都是定义了当运行这个image时要运行的命令

- ### CMD：制作者建议执行的默认命令

  > 容器启动时默认执行的命令

  ```dockerfile
  FROM centos
  ENV name Docker
  CMD echo "hello $name"
  ```

  > 如果docker run指定了其它命令,CMD会被忽略
  >
  > 如果定义多个CMD，只有最后一条会被执行。

  如果执行时输入`Docker run -it [image] /bin/bash`那么就不会执行CMD中的命令。

- ### ENTRYPOINT：服务运行的命令

  > 让容器以应用程序或者服务的形式运行
  >
  > <font color=#FF0033>不会被忽略，一定会执行</font>
  >
  > 最佳实践：写一个shell脚本作为entrypoint

  ```dockerfile
  COPY docker-entrypoint.sh /usr/local/bin
  ENTRYPOINT ["docker-entrypoint.sh"]
  
  EXPOSE 27017
  CMD ["mongod"]
  ```

  这个dockerfile是以exec的风格来写的，最重要的特点就是`[]`。是以运行文件或命令为单位而不是默认的bash命令。