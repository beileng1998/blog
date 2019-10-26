---
title: 代码云端部署框架 syncd 学习笔记【config】
date: 2019-10-20 17:00:00
tags: 
    - Go
    - syncd
---

## 项目配置描述 config.go
```go
type (
    // 总配置->子配置: 这样写利于开发
    Config struct {
        Serve   *ServeConfig
        Db      *DbConfig
        Log     *LogConfig
        Syncd   *SyncdConfig
        Mail    *MailConfig
    }

    SyncdConfig struct {
        LocalSpace      string
        RemoteSpace     string
        Cipher          string
        AppHost         string
    }

    LogConfig struct {
        Path    string
    }

    MailConfig struct {
        Enable  int
        Smtp    string
        Port    int
        User    string
        Pass    string
    }

    ServeConfig struct {
        Addr            string
        FeServeEnable   int
        ReadTimeout     int
        WriteTimeout    int
        IdleTimeout     int
    }

    DbConfig struct {
        Unix            string
        Host            string
        Port            int
        Charset         string
        User            string
        Pass            string
        DbName          string
        TablePrefix     string
        MaxIdleConns    int
        MaxOpenConns    int
        ConnMaxLifeTime int
    }
)

```

## 读配置

### 1 定义全局 configFile 结构

```go
import "github.com/Unknwon/goconfig"
var configFile *goconfig.ConfigFile
```
### 2 加载配置文件
```go
func initCfg() {
    var err error
    // 配置文件路径（这一步是为了检查路径，如果有错直接 crash）
    syncdIni := findSyncdIniFile()
    // 调用函数完成加载
    configFile, err = goconfig.LoadConfigFile(syncdIni)
    if err != nil {
        log.Fatalf("load config file failed, %s\n", err.Error())
    }
    outputInfo("Config Loaded", syncdIni)
}
// goconfig 包的加载函数
func LoadConfigFile(fileName string, moreFiles ...string) (c *ConfigFile, err error) {
	// Append files' name together.
	fileNames := make([]string, 1, len(moreFiles)+1)
	fileNames[0] = fileName
	if len(moreFiles) > 0 {
		fileNames = append(fileNames, moreFiles...)
	}

	c = newConfigFile(fileNames)

	for _, name := range fileNames {
		if err = c.loadFile(name); err != nil {
			return nil, err
		}
	}

	return c, nil
}
```
### 3 从文件取配置并赋值给配置描述结构体
```go
cfg := &syncd.Config{
        Serve: &syncd.ServeConfig{
            Addr: configOrDefault("serve", "addr", "8868"),
            FeServeEnable: configIntOrDefault("serve", "fe_serve_enable", 1),
            ReadTimeout: configIntOrDefault("serve", "read_timeout", 300),
            WriteTimeout: configIntOrDefault("serve", "write_timeout", 300),
            IdleTimeout: configIntOrDefault("serve", "idle_timeout", 300),
        },
        Db: &syncd.DbConfig{
            Unix: configOrDefault("database", "unix", ""),
            Host: configOrDefault("database", "host", ""),
            Port: configIntOrDefault("database", "port", 3306),
            Charset: "utf8mb4",
            User: configOrDefault("database", "user", ""),
            Pass: configOrDefault("database", "password", ""),
            DbName: configOrDefault("database", "dbname", ""),
            MaxIdleConns: configIntOrDefault("database", "max_idle_conns", 100),
            MaxOpenConns: configIntOrDefault("database", "max_open_conns", 200),
            ConnMaxLifeTime: configIntOrDefault("database", "conn_max_life_time", 500),
        },
        Log: &syncd.LogConfig{
            Path: configOrDefault("log", "path", "stdout"),
        },
        Syncd: &syncd.SyncdConfig{
            LocalSpace: configOrDefault("syncd", "local_space", "~/.syncd"),
            RemoteSpace: configOrDefault("syncd", "remote_space", "~/.syncd"),
            Cipher: configOrDefault("syncd", "cipher_key", ""),
            AppHost: configOrDefault("syncd", "app_host", ""),
        },
        Mail: &syncd.MailConfig{
            Enable: configIntOrDefault("mail", "enable", 0),
            Smtp: configOrDefault("mail", "smtp_host", ""),
            Port: configIntOrDefault("mail", "smtp_port", 465),
            User: configOrDefault("mail", "smtp_user", ""),
            Pass: configOrDefault("mail", "smtp_pass", ""),
        },
    }
```
```go
// User: configOrDefault("database", "user", "")
func configOrDefault(section, key, useDefault string) string {
    val, err := configFile.GetValue(section, key)
    if err != nil {
        return useDefault
    }
    return val
}
```

## 项目配置文件 syncd.ini
[] 表示 section  
格式：`key = value`
```ini
[syncd]
; 项目访问域名, 结尾不要加 `/`
app_host = http://localhost:8878

; 工作空间
local_space = /tmp/syncd_data

; 远端机器工作空间
remote_space = ~/.syncd

; AES加密/解密使用的私钥
; 秘钥需要进行base64编码
; 16 => AES-128, 24 => AES-192, 32 => AES-256
cipher_key = pG1L62EM0cPIIOwusQsbcV8Cs6j/M1RxLoXIZylWUC4=

[serve]
; HTTP服务监听的端口
addr = :8878

; 是否开启前端资源服务
; 开启后Syncd前端资源将不再依赖nginx等web服务
; 1 - 开启
; 0 - 关闭
fe_serve_enable = 1

; 读超时时间设置, 单位秒
read_timeout = 300

; 写超时时间设置, 单位秒
write_timeout = 300

; 空闲连接超时设置, 单位秒
idle_timeout = 300

[database]
; 数据库连接信息
; 必须是utf8mb4编码
;unix = 
;max_idle_conns = 100
;max_open_conns = 200
;conn_max_life_time = 500
host = 127.0.0.1
port = 3306
user = root
password = 123456
dbname = syncd

[log]
; 日志输出路径
; path = stdout - 打印到标准输出
; path = /path/file - 输出到文件
path = stdout

[mail]
; 是否开启邮件发送功能
; 0 - 关闭
; 1 - 开启
enable = 0

; 邮件smtp配置
smtp_host = smtp.exmail.qq.com
smtp_port = 465
smtp_user = 
smtp_pass = 
```