# Docker学习3：Docker&docker-machine

> 本文介绍docker的简单操作以及docker-machine的使用

## Docker-machine的使用：

> Docker-machine可以为用户建立一个安装好docker环境的虚拟机，并且可以直接在外部的命令行内访问虚拟机内的docker实例。

- docker-machine的安装：安装docker的时候就已经存在有docker-machine了
- docker-machine的使用：
  - 建立一个docker-machine：`docker-machine create machineName`
  - 查看本机中的docker-machine实例：`docker-machine ls`
  - 进入某个docker-machine实例中：`docker-machine ssh machineName`
  - 删除某个docker-machine实例：`docker-machine rm machineName` 
  - 暂停/开启某个docker-machine实例：`docker-machine stop/start machineName`
- docker-machine在外部命令行中的使用：
  - `docker-machine env machineName`
  - 然后按照提示输入指令，就将当前终端连接到了指定的docker-machine中