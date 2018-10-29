# Docker学习2：虚拟机自动管理-Vagrant

> vagrant的作用主要是在各种环境下为用户安装与操作（管理）稳定可靠的虚拟机，以下就是vagrant的安装过程。

## 1. vagrant的简单使用配置

1. 首先到vagrant的官网下载vagrant的软件，之后安装。之后就可以在命令行里面使用了。

2. 建立一个文件夹来装vagrant的配置文件，之后在终端界面cd到那个文件夹，使用命令`vagrant init centos/7`来建立一个centos/7的配置文件`Vagrantfile`

3. 之后其实就可以使用命令`vagrant up`来开启一个当前文件夹配置的centos/7虚拟机了。如果在本地就有这个虚拟机，就可以直接启动，如果没有也会自动去下载。

   > 要使用下载服务的话一定要注意需要有一个vagrant的账户，去官网注册，之后使用命令`vagrant login`来在本机注册，之后就可以正常使用了。

4. 要使用这台虚拟机，可以通过命令`vagrant ssh`来进入命令行界面来使用这台虚拟机

5. 常用命令：

   | 命令              | 注解                     |
   | ----------------- | ------------------------ |
   | `vagrant status`  | 查看当前运行的虚拟机状态 |
   | `vagrant halt`    | 暂停当前虚拟机           |
   | `vagrant destroy` | 删除当前机器             |

6. 通过配置vagrantfile可以建立各种不同的虚拟机，而我们可以从vagrant Cloud这个网站中找到相应的vagrantfile的简单配置。

## 2. vagrant的多种方便功能

> 使用vagrant建立虚拟机的过程中，可以通过配置它的配置文件Vagrantfile的方式来配置虚拟机的各种属性：

1. 网络属性

2. 硬件属性

3. 提前的配置脚本

   > 就是在安装虚拟机的时候就提前运行一些脚本制作一个比如已经配置好某些环境或安装好某些配置的虚拟机。

   ```bash
   config.vm.provision "shell",inline: <<-SHELL
     apt-get update
     apt-get install -y apache2
   SHELL
   ```

   通过这个功能可以使centos/7虚拟机在创建时就安装好docker:

   ```
   config.vm.provision "shell", inline: <<-SHELL
     sudo yum remove docker docker-common docker-selinux docker-engine
     sudo yum install -y yum-utils device-mapper-persistent-data lvm2
     sudo yum-config-manager -y --add-repo https://download.docker.com/linux/centos/docker-ce.repo
     sudo yum install -y docker-ce
     sudo systemctl start docker
   SHELL
   ```


