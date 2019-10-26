---
title: 代码云端部署框架 syncd 学习笔记【render】
date: 2019-10-22 15:30:00
tags: 
    - Go
    - syncd
---

## render.go

所有的返回值都是通过这一层来完成，这样就抽象出来一个中间层

实现了规范化和标准化，非常适合 rest

```go
// Copyright 2019 syncd Author. All Rights Reserved.
// Use of this source code is governed by a MIT-style
// license that can be found in the LICENSE file.

package render

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

const (
    CODE_OK = 0
    CODE_ERR_SYSTEM = 1000
    CODE_ERR_APP = 1001
    CODE_ERR_PARAM = 1002
	CODE_ERR_DATA_REPEAT = 1003
    CODE_ERR_LOGIN_FAILED = 1004
    CODE_ERR_NO_LOGIN = 1005
    CODE_ERR_NO_PRIV = 1006
    CODE_ERR_TASK_ERROR = 1007
	CODE_ERR_USER_OR_PASS_WRONG = 1008
	CODE_ERR_NO_DATA = 1009
)

func JSON(c *gin.Context, data interface{}) {
    c.JSON(http.StatusOK, gin.H{
        "code": CODE_OK,
        "message": "success",
        "data": data,
    })
}

func CustomerError(c *gin.Context, code int, message string) {
	c.JSON(http.StatusOK, gin.H{
        "code": code,
        "message": message,
    })
}

func RepeatError(c *gin.Context, message string) {
    c.JSON(http.StatusOK, gin.H{
        "code": CODE_ERR_DATA_REPEAT,
        "message": message,
    })
}

func NoDataError(c *gin.Context, message string) {
    c.JSON(http.StatusOK, gin.H{
        "code": CODE_ERR_NO_DATA,
        "message": message,
    })
}

func ParamError(c *gin.Context, message string) {
    c.JSON(http.StatusOK, gin.H{
        "code": CODE_ERR_PARAM,
        "message": message,
    })
}

func AppError(c *gin.Context, message string) {
    c.JSON(http.StatusOK, gin.H{
        "code": CODE_ERR_APP,
        "message": message,
    })
}

func Success(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "code": CODE_OK,
        "message": "success",
    })
}

```