---
title: docker 命令行参数文档
date: 2019-10-18 18:23:00
tags:
    - docker
---

## 主要参考
https://www.infoq.cn/article/KBTRC719-r6GHOPS3Cr8  
https://www.w3xue.com/manual/docker/

## 常用 Docker 命令列表
```
docker help—检查最新 Docker 可用命令；
docker attach—将本地输入、输出、错误流附加到正在运行的容器；
docker commit—从当前更改的容器状态创建新镜像；
docker exec—在活动或正在运行的容器中运行命令；
docker history—显示镜像历史记录；
docker info—显示系统范围信息；
docker inspect—查找有关 docker 容器和镜像的系统级信息；
docker login—登录到本地注册表或 Docker Hub；
docker pull—从本地注册表或 Docker Hub 中提取镜像或存储库；
docker ps—列出容器的各种属性；
docker restart—停止并启动容器；
docker rm—移除容器；
docker rmi—删除镜像；
docker run—在隔离容器中运行命令；
docker search—在 Docker Hub 中搜索镜像；
docker start—启动已停止的容器；
docker stop—停止运行容器；
docker version—提供 docker 版本信息。
```
## 一览表
```
容器生命周期管理命令
    run
    start/stop/restart
    kill
    rm
    pause/unpause
    create
    exec
容器操作命令
    ps
    inspect
    top
    attach
    events
    logs
    wait
    export
    port
容器rootfs命令
    commit
    cp
    diff
镜像仓库命令
    login
    pull
    push
    search
本地镜像管理命令
    images
    rmi
    tag
    build
    history
    save
    import
info|version命令
    info
    version
```

## docker run 详解

> 创建一个新的容器并运行一个命令

### 语法
```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### OPTIONS说明：
```
-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
-d: 后台运行容器，并返回容器ID；
-i: 以交互模式运行容器，通常与 -t 同时使用；
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
--name="nginx-lb": 为容器指定一个名称；
--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；
--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；
-h "mars": 指定容器的hostname；
-e username="ritchie": 设置环境变量；
--env-file=[]: 从指定文件读入环境变量；
--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；
-m :设置容器使用内存最大值；
--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
--link=[]: 添加链接到另一个容器；
--expose=[]: 开放一个端口或一组端口；
-v: 映射目录
```

### 实例
```
使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。

docker run --name mynginx -d nginx:latest
使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。

docker run -P -d nginx:latest
使用镜像nginx:latest以后台模式启动一个容器,将容器的80端口映射到主机的80端口,主机的目录/data映射到容器的/data。

docker run -p 80:80 -v /data:/data -d nginx:latest
使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。

w3xue@w3xue:~$ docker run -it nginx:latest /bin/bash
root@b8573233d675:/# 
```