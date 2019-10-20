---
title: 代码云端部署框架 syncd 学习笔记【dockerfile】
date: 2019-10-19 10:00:00
tags: 
    - Go
    - syncd
    - docker
---

## 官方文档
参考：https://docs.docker.com/engine/reference/builder/

通过编写 Dockerfile 同名文件，用户可在当前目录执行 `docker build` 来生成镜像

### 用法
命令通过 dockerfile 和上下文进行编译，上下文是文件的集合，通过指定文件系统路径或者 url（如 git 仓库） 可以确定包含的文件

上下文是递归处理的，对于文件系统路径，会递归包含子目录，对于 url, 会递归包含依赖模块

例子：对 pwd 进行编译

```sh
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```
注意：由于 build 是通过 daemon 执行，在启动的时候会将 context 发送到 daemon 进程，所以一般在空目录下放 Dockerfile 以及必须的文件进行编译，这样会提升性能

如果一定要放在项目中编译，可以使用 `.dockerignore` 来忽略多余内容

```
# comment
*/temp*
*/*/temp*
temp?
```
例子：通过任意 path 的 Dockerfile build, 使用参数 `"-f"`
```
$ docker build -f /path/to/a/Dockerfile .
```


### 格式
```
# Comment
INSTRUCTION arguments
```
A Dockerfile must start with a `FROM` instruction

### ENV 环境替换
例子：通过 ${foo} 代替 /bar
```dockerfile
FROM busybox
ENV foo /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux
```

### FROM 指定 base image
```dockerfile
FROM <image>[:<tag>] [AS <name>]

# 在 FROM 前定义参数 ARG
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
```

### RUN
The RUN instruction will execute any commands in a new layer on top of the current image and `commit the results`.
```dockerfile
RUN /bin/bash -c 'source $HOME/.bashrc
# or  
RUN ["/bin/bash", "-c", "echo hello"]
```

### CMD
三种形式：
- `CMD ["executable","param1","param2"]` (exec form, this is the preferred form)
- `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT)
- `CMD command param1 param2` (shell form)

一个 Dockerfile 只用有一个 CMD 起作用，如果写了多个，只有最后一个会起作用

If you would like your container to run the same executable every time, then you should consider using `ENTRYPOINT` in combination with `CMD`.

docker run 指定执行命令会覆盖 CMD

### LABEL
略

### EXPOSE
```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```
By default, EXPOSE assumes TCP. You can also specify UDP:
```dockerfile
EXPOSE 80/udp
```
To expose on both TCP and UDP, include two lines:
```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

```
docker run -p 80:80/tcp -p 80:80/udp ...
```

### ADD
The ADD instruction copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

两种形式:
- `ADD [--chown=<user>:<group>] <src>... <dest>`
- `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` (this form is required for paths containing whitespace)

```dockerfile
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```
### COPY
跟 ADD 差不多

### ENTRYPOINT
ENTRYPOINT has two forms:
- `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred)
- `ENTRYPOINT command param1 param2` (shell form)

An ENTRYPOINT allows you to configure a container that will run as an `executable`.

For example, the following will start nginx with its default content, listening on port 80:
```
docker run -i -t --rm -p 80:80 nginx
```
Command line arguments to docker run `<image>` will be appended after all elements in an exec form ENTRYPOINT, and will override all elements specified using CMD. This allows arguments to be passed to the entry point, i.e., docker run `<image>` -d will pass the -d argument to the entry point. You can override the ENTRYPOINT instruction using the docker run --entrypoint flag.

The shell form prevents any CMD or run command line arguments from being used, but has the disadvantage that your ENTRYPOINT will be started as a subcommand of /bin/sh -c, which does not pass signals. This means that the executable will not be the container’s PID 1 - and will not receive Unix signals - so your executable will not receive a SIGTERM from docker stop `<container>`.

有多个 ENTRYPOINT 时，最后一个 ENTRYPOINT 才会发挥作用

### VOLUME
```
VOLUME ["/data"]
```
The VOLUME instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers. The value can be a JSON array, VOLUME ["/var/log/"], or a plain string with multiple arguments, such as VOLUME /var/log or VOLUME /var/log /var/db.

### USER
TODO
### WORKDIR
TODO

## 本项目 Dockerfile
```dockerfile
# 第一个镜像：完成项目的 make
FROM golang:1.12-alpine3.10 AS build
COPY . /usr/local/src
WORKDIR /usr/local/src
RUN apk --no-cache add build-base && make

# 第二个镜像：使用第一个镜像的编译结果启动服务
FROM alpine:3.10
WORKDIR /syncd
COPY --from=build /usr/local/src/output /syncd
EXPOSE 8878
CMD [ "/syncd/bin/syncd" ]
```
