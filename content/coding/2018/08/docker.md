---
title: docker常用命令及配置 
date: 2018-08-13T11:29:55+08:00
author: Alvin
keywords: docker
---

## docker 常用命令

* 基本命令

```sh
docker images                       # 列出镜像
docker ps -a                        # 列出容器
docker rm  container_id             # 删除容器
docker rmi image_id                 # 删除镜像
docker tag image_id rep:tag         # 修改tag
docker commit container_id rep:tag  # 保存容器变更

docker save -o xx.img image_id      # 保存镜像
docker save image_id > xx.img       # 保存镜像
docker load < xx.img                # 加载镜像
```

* 创建并运行容器

```sh
# 在容器中运行bash
docker run -i -t rep:tag /bin/bash 

# -v 将本机目录映射到docker容器中
docker run -i -t -v /host_path/:/docker_path rep:tag /bin/bash
```

* 启动已存在的容器

```sh
docker start container_id && docker exec -it container_id /bin/bash
```

## 镜像加速

国内上 docker hub 较慢，可以用[阿里云镜像加速](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2F#/accelerator)。

* debian 8

修改 /etc/systemd/system/multi-user.target.wants/docker.service:

```sh
ExecStart: --registry-mirror=https://xxx.mirror.aliyuncs.commit
systemctl daemon-reload
systemctl restart docker
```

* mac

任务栏点击 docker 图标，-> Preferences -> Daemon -> Registry mirrors，列表中填上加速器地址即可。

* win10

docker 图标右键菜单选择 settings，左侧 daemon 导航栏，开启 Advanced 配置:

```json
"registry-mirrors": [
    "https://xxx.mirror.aliyuncs.com"
],
```

## centos 搭建 C/C++ 编译环境

* 获取 centos 镜像

```sh
docker pull centos
```

* 启动 centos 容器

```sh
docker run -it centos:latest /bin/bash
```

* C++ 编译环境

```sh
yum install gcc gcc-c++ make -y
yum install flex bison -y
yum install cmake autoconf automake binutils gettext libtool patch pkgconfig -y
yum install vim -y
yum install git -y
yum install expect -y
yum clean all
```
