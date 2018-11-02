# Docker学习4：Docker的image

> 对于docker的分层架构来说，image是其实现可扩展与小巧的重要元素。
>
> 什么是Image：文件与meta data的集合
>
> 可以分层，每一层都可以添加改变删除文件，成为一个新的image
>
> 不同的image可以共享相同的layer
>
> Image本身是read-only 的

![image-20181029112618169](/Users/haohao/Library/Application Support/typora-user-images/image-20181029112618169.png)

使用`sudo docker image ls`来查看当前系统存在的docker image。

## 通过Dockerfile来获取image

> 本节通过一个例子来讲解，我们要在已经有的ubuntu/14.04上建立一个redis-server的docker image

首先编辑dockerfile

```
FROM ubuntu:14.04                                      // 表示基于哪个image上构建
LABEL maintainer="Haohao"                              // 设置一个标记
RUN apt-get update && apt-get install -y redis-server  // image建立时运行的命令
EXPOSE 6379                                            // 要开启的端口
ENTRYPOINT [ "/usr/bin/redis-server" ]                 // 入口
```

之后就可以通过命令`docker build -t  haohao/redis:latest .`(在当前文件夹build一个docker image)来建立这个docker image

<font color=#FF0033>建立的时候会一步步执行dockerfile中的命令。</font>

## 通过registry来获取image

> registry可以说是image的github，我们可以在registry中pull喜欢的image，也可以将自己的image上传到registry

`sudo docker pull ubuntu:14.04`:这是一个获取ubuntu:14.04镜像的命令。

如果想获取其它镜像，可以登录dockerhub查找需要的image。

## 自己创建image

> 创建一个image，需要先编译好自己的服务，之后写Dockerfile分层搭建docker服务

例：要建立一个运行hello world的C程序作为base image

1. 首先在hello-world文件夹中编写一个c程序`hello.c`

   ```c
   #include<stdio.h>
   
   main(){
       printf("hello haohao");
   }
   ```

2. 之后使用命令`gcc -static hello.c -o hello`编译C程序为`hello`的可执行文件

3. 建立一个Dockerfile文件

   ```
   FROM scratch
   ADD hello /
   CMD ["/hello"]
   ```

`FROM scratch`：表示建立的是base image（基于scratch建立）

`ADD hello /`：向docker中的根路径加入hello这个可执行文件

`CMD ["/hello"]`：当加载这个docker时就执行根路径下的hello文件

4. 之后运行`docker build -t haohao95/hello-world`意思是建立一个tag为haohao95/hello-world的docker实例
5. 运行实例时`docker run haohao95/hello-world`

## 免除docker的sudo(将用户加入docker组)

1. 建立一个docker的组：`sudo groupadd docker`
2. 将vagrant用户加入docker：`sudo gpasswd -a vagrant docker`
3. 重启docker服务：`sudo service docker restart`
4. 之后重新连接ssh就可以不用sudo命令使用docker了