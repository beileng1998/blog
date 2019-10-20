---
title: 使用 go mod 作为包管理工具
date: 2019-10-16 16:52:00
tags: Go
---

## 步骤

### 修改代理为头条的 goproxy
可以加快速度，并解决部分（似乎有一些还是不行）包被墙的问题
```
vim ~/.bash_profile
// 添加一行
export GOPROXY="https://goproxy.io"
:wq
source ~/.bash_profile
```
### 使用 goland 新建项目选择 vgo

会自动创建 go.mod

写代码，然后 go mod tidy 即可
### 现有项目迁移

如果是现有项目，需要执行 go init 生成 go.mod，然后 go mod tidy

### golang 开头的包使用 github replace

需要手动修改 go.mod 文件规则
```
replace (
	golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
)

或者

replace golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a

```

> 注：mod 的包下载路径在：$GOPATH/pkg/mod/ 下面