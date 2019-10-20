---
title: 代码云端部署框架 syncd 学习笔记【build】
date: 2019-10-20 10:00:00
tags: 
    - Go
    - syncd
---

## 定义
```go
// 总的打包描述
type Build struct {
    repo        *Repo
    local       string
    // tmp 路径
    tmp         string
    packFile    string
    // 临时 .sh 脚本的名字，是一个随机生成的字符串
    scriptFile  string
    task        *command.Task
    result      *Result
}
// 代码仓库描述
type Repo struct {
    url     string
    branch  string
    commit  string
    local   string
}
// 打包任务描述
type Task struct {
    Commands    []string
    done        bool
    timeout     int
    termChan    chan int
    err         error
    result      []*TaskResult
}
// 打包结果描述
type Result struct {
    err     error
    status  int
    stime   int
    etime   int
}
```
## 流程

### 1 NewBuild

```go
func NewBuild(repo *Repo, local, tmp, packFile, scripts string) (*Build, error) {
    build := &Build{
        repo: repo,
        local: local,
        tmp: tmp,
        packFile: packFile,
        result: &Result{
            status: STATUS_INIT,
        },
    }
    // 根据初始化脚本新建项目相关的脚本文件 .sh
    if err := build.createScriptFile(scripts); err != nil {
        return build, err
    }
    build.initBuildTask()

    return build, nil
}
// 初始化打包 task
func (b *Build) initBuildTask() {
    // fetch 的 cmds
    cmds := b.repo.Fetch()
    cmds = append(cmds, []string{
        "echo \"Now is\" `date`",
        "echo \"Run user is\" `whoami`",
        fmt.Sprintf("rm -f %s", b.packFile),
        // 核心 cmd, 运行事先定义好的脚本文件
        fmt.Sprintf("/bin/bash -c %s", b.scriptFile),
        fmt.Sprintf("rm -f %s", b.scriptFile),
        fmt.Sprintf("rm -fr %s", b.local),
        "echo \"Compile completed\" `date`",
    }...)
    // 完成赋值
    b.task = command.NewTask(cmds, COMMAND_TIMEOUT)
}
```

### 2 Run

```go
func (b *Build) Run() {
    b.result.status = STATUS_ING
    b.result.stime = int(time.Now().Unix())
    b.task.Run()
    if err := b.task.GetError(); err != nil {
        b.result.status = STATUS_FAILED
        b.result.err = err
    } else {
        b.result.status = STATUS_DONE
    }
    b.result.etime = int(time.Now().Unix())
}
// task.Run()
func (t *Task) Run() {
    // 逐个执行 cmd
    for _, cmd := range t.Commands {
        result, err := t.next(cmd)
        t.result = append(t.result, result)
        if err != nil {
            t.err = errors.New("task run failed, " + err.Error())
            break
        }
    }
    t.done = true
}
```

### 3 Terminate
util/command 的 Run()

通过定义一个 chan 结合 select 来完成随时终止执行的操作
```go
type Command struct {
    Cmd             string
    Timeout         time.Duration
    // 需要终止时 <- 即可
    TerminateChan   chan int
    Setpgid         bool
    command         *exec.Cmd
    stdout          bytes.Buffer
    stderr          bytes.Buffer
}
func (c *Command) Run() error {
    if err := c.command.Start(); err != nil {
        return err
    }

    errChan := make(chan error)
    // 用另一个 goroutine 执行具体 cmd
    go func(){
        errChan <- c.command.Wait()
        defer close(errChan)
    }()

    var err error
    select {
    case err = <-errChan:   // cmd 完成了或者发生了错误
    case <-time.After(c.Timeout):   // 超时
        err = c.terminate()
        if err == nil {
            err = errors.New(fmt.Sprintf("cmd run timeout, cmd [%s], time[%v]", c.Cmd, c.Timeout))
        }
    case <-c.TerminateChan: // 用户终止
        err = c.terminate()
        if err == nil {
            err = errors.New(fmt.Sprintf("cmd is terminated, cmd [%s]", c.Cmd))
        }
    }
    return err
}
```