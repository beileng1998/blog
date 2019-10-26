---
title: 代码云端部署框架 syncd 学习笔记【合集 router】
date: 2019-10-22 18:00:00
tags: 
    - Go
    - syncd
---
## 项目地址

Github: https://github.com/dreamans/syncd

Gitee: https://gitee.com/dreamans/syncd

## 学习笔记 router
[Dockerfile 学习笔记](https://beileng1998.github.io/2019/10/19/Go/syncd/dockerfile/)

[Makefile 学习笔记](https://beileng1998.github.io/2019/10/19/Go/syncd/makefile/)

[build 学习笔记](https://beileng1998.github.io/2019/10/20/Go/syncd/build/)

[config 学习笔记](https://beileng1998.github.io/2019/10/20/Go/syncd/config/)

[gorm->mysql 学习笔记](https://beileng1998.github.io/2019/10/21/Go/syncd/db/)

[deploy 学习笔记](https://beileng1998.github.io/2019/10/21/Go/syncd/deploy/)

[model 学习笔记](https://beileng1998.github.io/2019/10/22/Go/syncd/model/)

[render 学习笔记](https://beileng1998.github.io/2019/10/22/Go/syncd/render/)

[install 学习笔记](https://beileng1998.github.io/2019/10/22/Go/syncd/install/)

[mail 学习笔记](https://beileng1998.github.io/2019/10/22/Go/syncd/mail/)
## 项目结构说明
```
.
├── Dockerfile  // 使用 docker 运行当前项目【学习编写 dockerfile】
├── Makefile    // make 编译文件【学习如何打包和编写 makefile】
├── build       // go 语言编写打包工具【学习定义并实现通用打包模块】
│   ├── build.go
│   ├── build_test.go
│   ├── repo.go
│   └── task.go
├── config.go   // 配置文件的描述 struct 【学习如何规范读写配置文件】
├── db.go       // 数据库统一连接工具库 【学习如何使用 gorm 连接 mysql】
├── deploy      // 抽象化的部署工具 【学习如何描述通用部署结构】
│   ├── deploy.go
│   ├── server.go
│   └── task.go
├── go.mod
├── go.sum
├── model       // 框架的数据模型与数据库交互 【学习如何设计模型与优雅使用 gorm 进行 crud】
│   ├── deploy_apply.go
│   ├── deploy_build.go
│   ├── deploy_task.go
│   ├── model.go
│   ├── project.go
│   ├── project_member.go
│   ├── project_space.go
│   ├── server.go
│   ├── server_group.go
│   ├── user.go
│   ├── user_role.go
│   └── user_token.go
├── module      // 框架的几个核心功能模块
│   ├── deploy
│   │   ├── apply.go
│   │   ├── build.go
│   │   └── deploy.go
│   ├── project
│   │   ├── member.go
│   │   ├── project.go
│   │   └── space.go
│   ├── server
│   │   ├── group.go
│   │   └── server.go
│   └── user
│       ├── login.go
│       ├── priv.go
│       ├── role.go
│       ├── token.go
│       └── user.go
├── public      // 打包好的静态前端文件
│   ├── css
│   ├── favicon.ico
│   ├── fonts
│   ├── img
│   ├── index.html
│   └── js
├── render      // 统一的渲染函数接口【学习规范 response】
│   └── render.go
├── resource    // 主要放了项目的数据库初始化 sql
│   └── sql
│       └── syncd_v2.0.0.sql
├── router      // 项目路由
│   ├── common
│   │   ├── hook.go
│   │   └── inspace.go
│   ├── deploy
│   │   ├── apply.go
│   │   ├── build.go
│   │   ├── deploy.go
│   │   └── mail.go
│   ├── middleware
│   │   └── api_priv.go
│   ├── project
│   │   ├── member.go
│   │   ├── project.go
│   │   └── space.go
│   ├── route
│   │   ├── api
│   │   │   └── api.go
│   │   └── route.go
│   ├── server
│   │   ├── group.go
│   │   └── server.go
│   └── user
│       ├── login.go
│       ├── my.go
│       ├── role.go
│       └── user.go
├── script      // 直接自动安装可执行文件的脚本【学习如何写 shell 脚本】
│   └── install.sh
├── sendmail.go // 发送邮件 【学习如何用 go 发送邮件】
├── syncd       // 项目入口文件【学习如何编写 gin 框架入口文件】
│   └── main.go
├── syncd.go    // 一些入口文件需要用到的函数
├── syncd.ini   // 配置文件
├── util        // 工具库【学习如何完成各个功能独立的 util】
│   ├── command
│   │   ├── command.go
│   │   ├── command_test.go
│   │   ├── task.go
│   │   └── task_test.go
│   ├── goaes
│   │   └── aes.go
│   ├── gofile
│   │   └── file.go
│   ├── gois
│   │   └── is.go
│   ├── golog
│   │   ├── file.go
│   │   └── log.go
│   ├── gopath
│   │   └── path.go
│   ├── goslice
│   │   └── slice.go
│   └── gostring
│       └── string.go
└── web

```
