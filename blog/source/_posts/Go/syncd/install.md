---
title: 代码云端部署框架 syncd 学习笔记【install】
date: 2019-10-22 15:30:00
tags: 
    - Go
    - syncd
    - sh
---

```sh
#!/bin/bash
# shell 变量
syncd_remote_repo=https://github.com/dreamans/syncd.git
syncd_repo_path=$HOME/.syncd-repo
# $0 shell 本身的文件名, install_path 是 sh 文件所在目录的下一级 syncd-deploy
syncd_install_path=$( cd `dirname $0`; pwd )/syncd-deploy

checkCommand() {
    # type go -> go is /usr/local/bin/go
    type $1 > /dev/null 2>&1
    # $? 为显示命令运行后的退出状态 0 表示没有错误
    # -ne 是否不相等
    if [ $? -ne 0 ]; then
        echo "error: $1 must be installed"
        echo "install exit"
        exit 1
    fi
}

# 确保这三个命令都是可运行的
checkCommand "go"
checkCommand "git"
checkCommand "make"

if [ -d ${syncd_install_path} ];then
    syncd_install_path=${syncd_install_path}-$( date +%Y%m%d%H%M%S )
fi
# 清空路径并 git clone
rm -fr ${syncd_repo_path}
git clone ${syncd_remote_repo} ${syncd_repo_path}

# make
cd ${syncd_repo_path}
make

# 把打包好的文件 cp 到目标路径
rm -fr ${syncd_install_path}
cp -r ${syncd_repo_path}/output ${syncd_install_path}

rm -fr ${syncd_repo_path}

cat << EOF

Installing Syncd Path:  ${syncd_install_path}
Install complete.

EOF
```