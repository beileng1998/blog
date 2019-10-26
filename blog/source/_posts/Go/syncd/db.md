---
title: 代码云端部署框架 syncd 学习笔记【gorm->mysql】
date: 2019-10-21 8:00:00
tags: 
    - Go
    - syncd
    - gorm
    - mysql
---

## 项目数据库连接 handler
```go
package syncd

import (
    "errors"
    "fmt"
    "time"
    // gorm 需要使用的 mysql 驱动
    _ "github.com/go-sql-driver/mysql"
    "github.com/jinzhu/gorm"
)

type DB struct {
    DbHandler   *gorm.DB
    cfg         *DbConfig
}

func NewDatabase(cfg *DbConfig) *DB {
    // 总配置分出来的数据库配置，复制到这里初始化
    return &DB{
        cfg: cfg,
    }
}

func (db *DB) Open() error {
    c, err := gorm.Open("mysql", db.parseConnConfig())
    if err != nil {
        return errors.New(fmt.Sprintf("mysql connect failed, %s", err.Error()))
    }

    c.LogMode(false)
    c.DB().SetMaxIdleConns(db.cfg.MaxIdleConns)
    c.DB().SetMaxOpenConns(db.cfg.MaxOpenConns)
    c.DB().SetConnMaxLifetime(time.Second * time.Duration(db.cfg.ConnMaxLifeTime))

    db.DbHandler = c
    return nil
}

func (db *DB) Close() {
    db.DbHandler.Close()
}

func (db *DB) parseConnConfig() string {
    var connHost string
    if db.cfg.Unix != "" {
        connHost = fmt.Sprintf("unix(%s)", db.cfg.Unix)
    } else {
        connHost = fmt.Sprintf("tcp(%s:%d)", db.cfg.Host, db.cfg.Port)
    }
    s := fmt.Sprintf("%s:%s@%s/%s?charset=%s&parseTime=True&loc=Local", db.cfg.User, db.cfg.Pass, connHost, db.cfg.DbName, db.cfg.Charset)
    return s
}

```