# Gin学习笔记


<!--more-->

### 路由

``` 
import (
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	user := r.Group("/user")
	user.GET("/index", func(c *gin.Context) {})
	user.POST("/login", func(c *gin.Context) {})
	r.Run()
}
```

### 获取参数

1. Query方式
   
获取URL中？后面所携带的参数，例如/name=admin&pwd=123456

``` 
r.GET("/", func(c *gin.Context) {
	name := c.Query("name")
	pwd := c.Query("pwd")
	
	c.JSON(http.StatusOK, gin.H{
		"name": name,
		"pwd":  pwd,
	})
})
```
2. Form方式

``` 
r.POST("/", func(c *gin.Context) {
	username := c.PostForm("username") //对应h5表单中的name字段
	password := c.PostForm("password")
	c.HTML(http.StatusOK, "index.html", gin.H{
		"username": username,
		"password": password,
	})
})
```

3. Path方式

通过URL路径传递，例如/user/admin

``` 
r.GET("/user/:username", func(c *gin.Context) {
    username := c.Param("username")
    c.JSON(http.StatusOK, gin.H{
         "username": username,
    })
})
```

4. Body

``` 
    router.POST("/post-data", func(c *gin.Context) {
 
        var inputData struct {
            Name  string `json:"name"`
            Email string `json:"email"`
        }

        if err := c.ShouldBindJSON(&inputData); err != nil {
            c.JSON(400, gin.H{"error": err.Error()})
            return
        }

        name := inputData.Name
        email := inputData.Email

        c.JSON(200, gin.H{"name": name, "email": email})
    })
```

### HTML渲染

使用LoadHTMLGlob() 或者 LoadHTMLFiles()来渲染HTML模板

``` 
func main() {
	r := gin.Default()
	r.LoadHTMLGlob("./templates/*")
	r.GET("/demo",func(c *gin.Context) {
		c.HTML(http.StatusOK,"index.html",gin.H{
			"name":"admin",
			"pwd":"123456",
		})
	})
	r.Run()
}
```

需要创建一个存放模板文件的templates文件夹，然后在其内部写入一个index.html。

### Tips


1. api设计时注重实际需求，比如用户登陆用手机号不用id。
2. CURD中 query update delete 都可以把id放进url里不放form里

3. 使用【import _ 包路径】只是引用该包，仅仅是为了调用init()函数，所以无法通过包名来调用包中的其他函数。
4. 指针函数

```
func (group *RouterGroup) POST(relativePath string, handlers ...HandlerFunc) IRoutes {
    return group.handle(http.MethodPost, relativePath, handlers)
}

 // HandlerFunc defines the handler used by gin middleware as return value.
type HandlerFunc func(*Context)


func Run(router *gin.Engine) {
    user := router.Group("app/user")
    user.POST("/register", buildPolicyHolder)

    router.Run()
}
```

5. 存储用户密码

加密方式: 密码 + 盐(一串随机数) + 用户信息 + Hash (Hash Table自己创建)
数据库存储用户密码和盐。
      

      



