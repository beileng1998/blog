---
title: 使用 docker 的一些例子
date: 2019-10-18 11:50:33
tags:
    - docker
    - mysql
    - tools
---

## 本地安装 portainerUI
```
docker volume create portainer_data

docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

## 本地运行 mysql docker 服务
```
mkdir {anypath}/mysql

docker run -d -p 3306:3306 --restart always --privileged=true --name my_mysql -e MYSQL_ROOT_PASSWORD=123456 -v={anypath}/mysql:/mysql/data mysql
```