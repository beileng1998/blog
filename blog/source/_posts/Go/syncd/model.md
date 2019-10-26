---
title: 代码云端部署框架 syncd 学习笔记【model】
date: 2019-10-22 15:00:00
tags: 
    - Go
    - syncd
    - gorm
---
## 核心 crud -> model.go
```go
// 单个 field 的查询条件
type WhereParam struct {
    Field   string
    Tag     string
    Prepare interface{}
}

// 对一次查询条件的总描述
type QueryParam struct {
    Fields     string
    Offset     int
    Limit      int
    Order      string
    Where      []WhereParam
}

// 统一的建表 api
func Create(model interface{}) bool {
    db := syncd.App.DB.DbHandler.Create(model)
    if err := db.Error; err != nil {
        syncd.App.Logger.Warning("mysql execute error: %s, sql [%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func GetMulti(model interface{}, query QueryParam) bool {
    db := syncd.App.DB.DbHandler.Offset(query.Offset)
    if query.Limit > 0 {
        db = db.Limit(query.Limit)
    }
    if query.Fields != "" {
        db = db.Select(query.Fields)
    }
    if query.Order != "" {
        db = db.Order(query.Order)
    }
    db = parseWhereParam(db, query.Where)
    // 把条件都组装好了后开始 find
    db.Find(model)
    if err := db.Error; err != nil {
        syncd.App.Logger.Warning("mysql query error: %s, sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func Count(model interface{}, count *int, query QueryParam) bool {
    db := syncd.App.DB.DbHandler.Model(model)
    db = parseWhereParam(db, query.Where)
    db = db.Count(count)
    if err := db.Error; err != nil {
        syncd.App.Logger.Warning("mysql query error: %s, sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func Delete(model interface{}, query QueryParam) bool {
    if len(query.Where) == 0 {
        syncd.App.Logger.Warning("mysql query error: delete failed, where conditions cannot be empty")
        return false
    }
    db := syncd.App.DB.DbHandler.Model(model)
    db = parseWhereParam(db, query.Where)
    db = db.Delete(model)
    if err := db.Error; err != nil {
        syncd.App.Logger.Warning("mysql query error: %s, sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func DeleteByPk(model interface{}) bool {
    db := syncd.App.DB.DbHandler.Model(model)
    db.Delete(model)
    if err := db.Error; err != nil {
        syncd.App.Logger.Warning("mysql query error: %s, sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func GetOne(model interface{}, query QueryParam) bool {
    db := syncd.App.DB.DbHandler.Model(model)
    if query.Fields != "" {
        db = db.Select(query.Fields)
    }
    db = parseWhereParam(db, query.Where)
    db = db.First(model)
    if err := db.Error; err != nil && !db.RecordNotFound() {
        syncd.App.Logger.Warning("mysql query error: %s, sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func GetByPk(model interface{}, id interface{}) bool {
    db := syncd.App.DB.DbHandler.Model(model)
    db.First(model, id)
    if err := db.Error; err != nil && !db.RecordNotFound() {
        syncd.App.Logger.Warning("mysql query error: %s sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func UpdateByPk(model interface{}) bool {
    db := syncd.App.DB.DbHandler.Model(model)
    db = db.Updates(model)
    if err := db.Error; err != nil {
        syncd.App.Logger.Warning("mysql query error: %s, sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

func Update(model interface{}, data interface{}, query QueryParam) bool {
    db := syncd.App.DB.DbHandler.Model(model)
    db = parseWhereParam(db, query.Where)
    db = db.Updates(data)
    if err := db.Error; err != nil {
        syncd.App.Logger.Warning("mysql query error: %s, sql[%v]", err.Error(), db.QueryExpr())
        return false
    }
    return true
}

// 把所有 param 组装一个 single sql
func parseWhereParam(db *gorm.DB, where []WhereParam) *gorm.DB {
    if len(where) == 0 {
        return db
    }
    var (
        plain []string
        prepare []interface{}
    )
    for _, w := range where {
        tag := w.Tag
        if tag == "" {
            tag = "="
        }
        var plainFmt string
        switch tag {
        case "IN":
            plainFmt = fmt.Sprintf("%s IN (?)", w.Field)
        default:
            plainFmt = fmt.Sprintf("%s %s ?", w.Field, tag)
        }
        plain = append(plain, plainFmt)
        prepare = append(prepare, w.Prepare)
    }
    return db.Where(strings.Join(plain, " AND "), prepare...)
}

```

## user
包括三个表：
- user(user 的基本信息)
- user_role(user 的权限信息)
- user_token(user 的认证信息)

```go
// user.go
type User struct {
    ID              int         `gorm:"primary_key"`
    RoleId          int         `gorm:"type:int(11);not null;default:0"`
    Username	    string      `gorm:"type:varchar(20);not null;default:''"`
    Password        string      `gorm:"type:char(32);not null;default:''"`
    Salt            string      `gorm:"type:char(10);not null;default:''"`
    Truename        string      `gorm:"type:varchar(20);not null;default:''"`
    Mobile          string      `gorm:"type:varchar(20);not null;default:''"`
    Email           string      `gorm:"type:varchar(500);not null;default:''"`
    Status          int         `gorm:"type:int(11);not null;default:0"`
    LastLoginTime   int         `gorm:"type:int(11);not null;default:0"`
    LastLoginIp     string      `gorm:"type:varchar(50);not null;default:''"`
    Ctime           int         `gorm:"type:int(11);not null;default:0"`
}

func (m *User) TableName() string {
    return "syd_user"
}

func (m *User) Create() bool {
    m.Ctime = int(time.Now().Unix())
    return Create(m)
}

func (m *User) UpdateByFields(data map[string]interface{}, query QueryParam) bool {
    return Update(m, data, query)
}

func (m *User) List(query QueryParam) ([]User, bool) {
    var data []User
    ok := GetMulti(&data, query)
    return data, ok
}

func (m *User) Count(query QueryParam) (int, bool) {
    var count int
    ok := Count(m, &count, query)
    return count, ok
}

func (m *User) Delete() bool {
    return DeleteByPk(m)
}

func (m *User) Get(id int) bool {
    return GetByPk(m, id)
}

func (m *User) GetOne(query QueryParam) bool {
	return GetOne(m, query)
}
```

```go
// user_role.go
type UserRole struct {
    ID          int         `gorm:"primary_key"`
    Name        string      `gorm:"type:varchar(100);not null;default:''"`
    Privilege   string	    `gorm:"type:varchar(2000);not null;default:''"`
    Ctime       int         `gorm:"type:int(11);not null;default:0"`
}
```

```go
// user_token.go
type UserToken struct {
    ID              int         `gorm:"primary_key"`
    UserId          int         `gorm:"type:int(11);not null;default:0"`
    Token           string      `gorm:"type:varchar(100);not null;default:''"`
    Expire          int         `gorm:"type:int(11);not null;default:0"`
    Ctime           int         `gorm:"type:int(11);not null;default:0"`
}
```

其余 model 略