# Docker学习7：搭建私有的registry服务

> 使用简单的`docker push`命令可以将建立的docker image提交到云上。但有时候为了安全与隐私需要在自己的服务器上建立一个私人的registry服务。

## 1. 服务器上搭建registry服务

> 主要就是在服务器上建立一个registry的container
>
> [安装registry](https://hub.docker.com/_/registry/)

```bash
$ docker run -d -p 5000:5000 --restart always --name registry registry:2
```



## 2. 与私人registry进行连接

例：

- 建立一个带有registry地址Tag的docker image

  ```bash
  docker build -t 149.28.91.139:5000/hello-world .
  ```

- 之后要将我们的服务器加入可信任列表中

  ```bash
  sudo vim /etc/docker/deamon.json
  ```

  ```json
  { "insecure-registries":["149.28.91.139:5000"] }
  ```

  ```bash
  sudo vim /lib/systemd/system/docker.service
  ```

  在`[Service]`项中加入一项

  ```
  EnvironmentFile=-/etc/docker/daemon.json
  ```

- 之后就可以push到私人的registry中去了

  ```bash
  docker push 149.28.91.139:5000/hello-world
  ```

- 查看registry中存在的docker

  ```
  149.28.91.139:5000/v2/_catalog
  ```

- pull回我们存储在私人registry的image

  ```bash
  docker pull 149.28.91.139:5000/hello-world
  ```
