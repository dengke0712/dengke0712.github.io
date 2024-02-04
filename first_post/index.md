# GORM学习总结




#### ORM

Object Relational Mapping(对象关系映射)，其主要作用是在编程中，把面向对象的概念跟数据库中表的概念对应起来。举例来说，定义一个对象，那就对应着一张表，这个对象的实例，就对应着表中的一条记录。

GORM（Go Object Relational Mapping）用于将Go应用程序中的结构体与关系型数据库之间进行映射，从而简化了数据库操作。它提供了一种便捷的方式来执行常见的数据库操作，如创建、读取、更新和删除（CRUD），以及复杂的查询和关联操作。

#### 声明模型
```
type User struct {
    ID   uint
    Name string
    Age  int
}
```

使用声明模型创建数据库表
```
db.AutoMigrate(&User{})
```

如果模型是动态模型，可使用原生SQL创建数据库表。
```
db.Exec()
```


#### CREATE

<u>传入数据的指针</u>
```
newUser := User{Name: "John", Age: 30}
db.Create(&newUser)
```
根据Map创建
```
db.Model(&User{}).Create(map[string]interface{}{
"Name": "jinzhu", "Age": 18,
})

db.Model(&User{}).Create([]map[string]interface{}{
{"Name": "jinzhu_1", "Age": 18},
{"Name": "jinzhu_2", "Age": 20},
})
```

#### QUERY

通过主键提取记录
```
var user User
db.First(&user, 1)
```
提取所有记录
```
var users []User
db.Find(&users)
```

按条件查询

```
db.Where("Age > ?", 25).Find(&users)
db.Where("Name LIKE ?", "%John%").Find(&users)
```
查询名字中<u>包含</u>John的用户

#### UPDATE

```
db.Model(&user).Update("Age", 31)
```
更新多列
```
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
```
使用 struct 更新时，GORM 只更新非零字段。需要使用 map 来更新，或使用 Select 来指定要更新的字段。

#### DELETE
```
db.Delete(&user)
db.Where("name = ?", "jinzhu").Delete(&user)
```
根据主键批量删除
```
db.Delete(&users, []int{1,2,3})
```

#### Sorting & Limiting

```
db.Order("Age desc").Limit(10).Find(&users)
```

#### 关联

```
type User struct {
  gorm.Model
  Name         string
  CompanyRefer int
  Company      Company `gorm:"foreignKey:CompanyRefer;constraint:OnUpdate:CASCADE,OnDelete:SET NULL;"`
}

type Company struct {
  ID   int
  Name string
}
```

#### Transactions
```
tx := db.Begin()
if err := tx.Create(&User{Name: "Alice"}).Error; err != nil {
    tx.Rollback()
} else {
    tx.Commit()
}
```

使用**tx**而不能使用db，否则无效。

#### Tips

初始化指针，防止指针为空。

```
var city = &entity.City{}
```

检查是否存在、不存在则创建，要考虑全面。

```
if errors.Is(result.Error, gorm.ErrRecordNotFound) || city == nil {
    tx := data.DB.Create(&jsonCity)
}
```

