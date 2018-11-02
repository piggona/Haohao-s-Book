# Docker学习5：Container

> 什么是container:
>
> - 通过image创建
> - 在Image layer（只读）之上建立一个container layer（可读写）
> - 类比面向对象：类&实例
> - Image负责app的存储与分发，container负责操作app

## 交互式运行一个container

> 运行一个container就使用命令：`docker run imageTag`

> 使用命令`docker container ls`来查看现在运行的container。

<font color=#FF0033>但有时候会发现查看不到刚才运行的container,是因为使用`docker run`命令就会执行Dockerfile中的`CMD`项中的命令。但如果命令没有指定维持状态，就会立即停止。</font>

例：

使用centos的image,执行它所运行的命令是`/bin/bash`

直接运行`docker run centos`就会立即执行停止，无法进入bash界面。

要保持bash的运行状态，就使用`docker run -it centos`,这样就会进入交互的界面。

## 选择想要的image或container进行操作

> 我们对某个image或某个container进行操作时，总是需要通过image或container的id进行操作，有没有什么办法可以方便的取出我们想要的image&container

> 想了解docker的操作命令：`docker --help` `docker run --help`

### 1. 查询所有image/container只显示id

```bash
docker container/image ls -aq
```

### 2. 将满足条件的image/container全部删除（也可以是其它命令）

```
docker rm $(docker container ls -f "status=exited" -q)
```

## docker container commit

> 基于已经存在的container建立一个image

```bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]] [flags]
```

## docker image build

> 根据Dockerfile安全的建立一个image

```bash
docker build -t haohao/centos-vim .
```

-t选项：是建立新的image的tag

后面是docker寻找Dockerfile文件的路径。