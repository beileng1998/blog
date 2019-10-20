---
title: 代码云端部署框架 syncd 学习笔记【makefile】
date: 2019-10-19 15:00:00
tags: 
    - Go
    - syncd
    - makefile
---
## 本项目的 makefile
```makefile
# 定义常量
GO_CMD=go
SYNCD_BIN=syncd_bin
SYNCD_BIN_PATH=./output/bin
SYNCD_ETC_PATH=./output/etc
SYNCD_PUBLIC_PATH=./output/public
SYNCD_LOG_PATH=./output/log
SYNCD_RES_PATH=./output/resource

# make 默认执行
.PHONY: all
all: clean build install

# make linux
.PHONY: linux
linux: clean build-linux install

# 默认编译
.PHONY: build
build:
	@echo "build syncd start >>>"
    # 指定代理以及拉取依赖
	GOPROXY=https://goproxy.io $(GO_CMD) mod tidy
    # 根据 main.go 编译出 SYNCD_BIN
	$(GO_CMD) build -o $(SYNCD_BIN) ./syncd/main.go
	@echo ">>> build syncd complete"

# 打包（主要是处理静态文件、配置文件）
.PHONY: install
install:
	@echo "install syncd start >>>"
	mkdir -p $(SYNCD_BIN_PATH)
    # 将编译好的 SYNCD_BIN 移动到目录并重命名为 syncd
	mv $(SYNCD_BIN) $(SYNCD_BIN_PATH)/syncd
	mkdir -p $(SYNCD_ETC_PATH)
    # 移动配置文件
	cp ./syncd.ini $(SYNCD_ETC_PATH)
    # 复制静态文件到指定目录
	cp -r ./public $(SYNCD_PUBLIC_PATH)
	mkdir -p $(SYNCD_LOG_PATH)
    # 复制资源文件到指定目录
	cp -r ./resource $(SYNCD_RES_PATH)
	@echo ">>> install syncd complete"

.PHONY: clean
clean:
	@echo "clean start >>>"
	rm -fr ./output
	rm -f $(SYNCD_BIN)
	@echo ">>> clean complete"

.PHONY: build-linux
build-linux:
	@echo "build-linux start >>>"
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 $(GO_CMD) build -o $(SYNCD_BIN) ./syncd/main.go
	@echo ">>> build-linux complete"
```

## 学习 MakeFile
参考 https://seisman.github.io/how-to-write-makefile/introduction.html

### MakeFile 的规则
```makefile
target ... : prerequisites ...
    command
    ...
    ...
```
#### target
可以是一个 object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）
#### prerequisites
生成该 target 所依赖的文件
#### command
该 target 要执行的命令（任意的shell命令）

> 核心概念：prerequisites 中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。

#### 伪目标 .PHONY
除了第一个目标是默认执行，其他标记了伪目标的需要指定才会执行，不生成目标文件