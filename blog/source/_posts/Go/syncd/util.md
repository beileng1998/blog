---
title: 代码云端部署框架 syncd 学习笔记【util】
date: 2019-10-22 17:30:00
tags: 
    - Go
    - syncd
---

学习 syncd 的工具箱怎么实现~

## 如何用 go 执行一个 cmd
```go
// Copyright 2018 syncd Author. All Rights Reserved.
// Use of this source code is governed by a MIT-style
// license that can be found in the LICENSE file.

package command

import (
    "bytes"
    "errors"
    "fmt"
    "os/exec"
    "strings"
    "syscall"
    "time"
)

const (
    DEFAULT_RUM_TIMEOUT = 3600
)
// 一个完整的 cmd 描述
type Command struct {
    Cmd             string
    Timeout         time.Duration
    // 这个是一个无缓冲 chan, 用来在任意时刻终止 cmd
    TerminateChan   chan int
    Setpgid         bool
    command         *exec.Cmd
    // 通过 buffer 来进行输出
    stdout          bytes.Buffer
    stderr          bytes.Buffer
}

func NewCmd(c *Command) (*Command, error) {
    if c.Timeout == 0 * time.Second {
        c.Timeout = DEFAULT_RUM_TIMEOUT * time.Second
    }
    if c.TerminateChan == nil {
        c.TerminateChan = make(chan int)
    }
    cmd := exec.Command("/bin/bash", "-c", c.Cmd)
    if c.Setpgid {
        cmd.SysProcAttr = &syscall.SysProcAttr{Setpgid: true}
    }
    cmd.Stderr = &c.stderr
    cmd.Stdout = &c.stdout
    c.command = cmd

    return c, nil
}

func (c *Command) Run() error {
    if err := c.command.Start(); err != nil {
        return err
    }

    errChan := make(chan error)
    go func(){
        errChan <- c.command.Wait()
        defer close(errChan)
    }()

    var err error
    // 在运行过程中三者都是阻塞的，正常情况下 errChan 会最先接通，然后返回
    select {
    case err = <-errChan:
    case <-time.After(c.Timeout):
        err = c.terminate()
        if err == nil {
            err = errors.New(fmt.Sprintf("cmd run timeout, cmd [%s], time[%v]", c.Cmd, c.Timeout))
        }
    case <-c.TerminateChan:
        err = c.terminate()
        if err == nil {
            err = errors.New(fmt.Sprintf("cmd is terminated, cmd [%s]", c.Cmd))
        }
    }
    return err
}

func (c *Command) Stderr() string {
    return strings.TrimSpace(string(c.stderr.Bytes()))
}

func (c *Command) Stdout() string {
    return strings.TrimSpace(string(c.stdout.Bytes()))
}

func (c *Command) terminate() error {
    if c.Setpgid {
        return syscall.Kill(-c.command.Process.Pid, syscall.SIGKILL)
    } else {
        return syscall.Kill(c.command.Process.Pid, syscall.SIGKILL)
    }
}
```

## 如何用 go 进行加密
```go
package goaes

import (
    "bytes"
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "errors"
    "io"
)

func Encrypt(key []byte, text []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }

    msg := pad(text)
    ciphertext := make([]byte, aes.BlockSize+len(msg))
    iv := ciphertext[:aes.BlockSize]
    if _, err := io.ReadFull(rand.Reader, iv); err != nil {
        return nil, err
    }

    cfb := cipher.NewCFBEncrypter(block, iv)
    cfb.XORKeyStream(ciphertext[aes.BlockSize:], []byte(msg))

    return ciphertext, nil
}

func Decrypt(key []byte, text []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    if (len(text) % aes.BlockSize) != 0 {
        return nil, errors.New("blocksize must be multipe of decoded message length")
    }
    iv := text[:aes.BlockSize]
    msg := text[aes.BlockSize:]

    cfb := cipher.NewCFBDecrypter(block, iv)
    cfb.XORKeyStream(msg, msg)

    unpadMsg, err := unpad(msg)
    if err != nil {
        return nil, err
    }

    return unpadMsg, nil
}

func pad(src []byte) []byte {
    padding := aes.BlockSize - len(src)%aes.BlockSize
    padtext := bytes.Repeat([]byte{byte(padding)}, padding)
    return append(src, padtext...)
}

func unpad(src []byte) ([]byte, error) {
    length := len(src)
    unpadding := int(src[length-1])

    if unpadding > length {
        return nil, errors.New("unpad error. This could happen when incorrect encryption key is used")
    }

    return src[:(length - unpadding)], nil
}
```