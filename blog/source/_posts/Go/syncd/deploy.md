---
title: 代码云端部署框架 syncd 学习笔记【deploy】
date: 2019-10-21 15:00:00
tags: 
    - Go
    - syncd
---
## deploy 描述
```go
type Deploy struct {
    ID              int
    User            string
    // 部署前运行命令
    PreCmd          string
    // 部署后运行命令
    PostCmd         string
    // 部署路径
    DeployPath      string
    DeployTmpPath   string
    PackFile        string
    // 部署到的服务器描述（可以包括多台->分布式）
    // 每个服务器描述都包含一个 task 描述
    srvs            []*Server
    status          int
    // 每个服务器开始部署前 +1, defer wg.Done(), 利用这个可以同步部署状态
    wg              sync.WaitGroup
}

type Server struct {
    ID              int
    Addr            string
    User            string
    Port            int
    PreCmd          string
    PostCmd         string
    Key             string
    PackFile        string
    DeployTmpPath   string
    DeployPath      string
    // 每个 server 都有一个 task 要执行
    task            *command.Task
    result          *ServerResult
}
```