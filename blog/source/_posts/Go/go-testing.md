---
title: Go 单元测试模板 testing 使用
date: 2019-10-16 11:40:00
tags: Go
---

## 基本逻辑
使用 testing 包，定义测试函数，参数为 ```t *testing.T```，通过 ```t.Run()```传入函数和参数执行，通过 ```t.Errorf()``` 来抛出错误

### 测试多个 case
```go
package test

import (
    "github.com/dreamans/syncd/util/gostring"
    "testing"
)

func TestJoinStrings(t *testing.T) {
    type args []string
    tests := []struct {
        name    string  // 用例名字
        args    args    // 传给被测函数的参数
        want    string  // 预期返回结果
        wantErr bool    // 用bool方便判断是否返回error，如果类型改为error反而不好判断
    }{
        {"case 0", args{"a", "b", "c"}, "abc", false},
        {"case 1", args{}, "", false},
        {"case_2", args{" a", "b "}, " ab ", false},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := gostring.JoinStrings(tt.args...)
            if got != tt.want {
                t.Errorf("Division() = %v, want %v", got, tt.want)
            }
        })
    }
}

```
### 压力测试与计时示例
```go
func Benchmark_JoinStrings(b *testing.B) {
    for i := 0; i < b.N; i++ { // use b.N for looping
        gostring.JoinStrings("a", "b", "c")
    }
}

func Benchmark_TimeConsumingFunction(b *testing.B) {
    b.StopTimer() // 调用该函数停止压力测试的时间计数

    // 此处做一些初始化的工作,例如读取文件数据,数据库连接之类的,
    // 这样这些时间不影响我们测试函数本身的性能

    b.StartTimer() // 重新开始时间

    b.N=1234 // 自定义执行1234次

    for i := 0; i < b.N; i++ {
        gostring.JoinStrings("a", "b", "c")
    }
    // 运行结束后会自动打印出时间
}
```