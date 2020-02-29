---
title: 高性能的 Go Web 框架 - gin
categories: 
- collect
tags:
- Go
date: 2018-07-26 20:36:25
---



## Gin 的使用

### 安装和更新

首次安装，使用 `go get`命令获取即可。

```sh
$ go get github.com/gin-gonic/gin
```

更新就是常规的 `go get -u`。

```sh
$ go get -u github.com/gin-gonic/gin
```

### 快速运行

在你的 main 包中，引入 gin 包并初始化。

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	// 初始化引擎
	engine := gin.Default()
	// 注册一个路由和处理函数
	engine.Any("/", WebRoot)
	// 绑定端口，然后启动应用
	engine.Run(":9205")
}

/**
* 根请求处理函数
* 所有本次请求相关的方法都在 context 中，完美
* 输出响应 hello, world
*/
func WebRoot(context *gin.Context) {
	context.String(http.StatusOK, "hello, world")
}
```

一个最简单的应用就写好了，来运行下试试:

```sh
$ go run
[GIN-debug] Listening and serving HTTP on :9205
```

访问 [http://127.0.0.1:9205](http://127.0.0.1:9205/) ，就可以得到响应 “hello, world” 。

## 路由 (Router)

### Restful Api

你可以注册路由方法有 GET, POST, PUT, PATCH, DELETE 和 OPTIONS.

使用很简单，直接调用同名的方法即可。

```go
// 省略的代码 ...

func main() {
	router := gin.Default()

	router.GET("/someGet", getting)
	router.POST("/somePost", posting)
	router.PUT("/somePut", putting)
	router.DELETE("/someDelete", deleting)
	router.PATCH("/somePatch", patching)
	router.HEAD("/someHead", head)
	router.OPTIONS("/someOptions", options)

	// 默认绑定 :8080
	router.Run()
}

// 省略的代码 ...
```

### 动态路由（参数路由）

有时候我们需要动态的路由，如 `/user/:id`，通过调用的 url 来传入不同的 id . 在 Gin 中很容易处理这种路由：

```go
// 省略的代码 ...

func main() {
	router := gin.Default()

	// 注册一个动态路由
  	// 可以匹配 /user/leeifme
  	// 不能匹配 /user 和 /user/
	router.GET("/user/:name", func(c *gin.Context) {
		// 使用 c.Param(key) 获取 url 参数
		name := c.Param("name")
		c.String(http.StatusOK, "Hello %s", name)
	})

  	// 注册一个高级的动态路由
	// 该路由会匹配 /user/leeifme/ 和 /user/leeifme/send
	// 如果没有任何路由匹配到 /user/leeifme, 那么他就会重定向到 /user/leeifme/，从而被该方法匹配到
	router.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
		message := name + " is " + action
		c.String(http.StatusOK, message)
	})

	router.Run(":8080")
}

// 省略的代码 ...
```

### 路由组

一些情况下，我们会有统一前缀的 url 的需求，典型的如 Api 接口版本号 `/v1/something`。Gin 可以使用 Group 方法统一归类到路由组中：

```go
// 省略的代码 ...

func main() {
	router := gin.Default()

	// 定义一个组前缀
  	// /v1/login 就会匹配到这个组
	v1 := router.Group("/v1")
	{
		v1.POST("/login", loginEndpoint)
		v1.POST("/submit", submitEndpoint)
		v1.POST("/read", readEndpoint)
	}

	// 定义一个组前缀
  	// 不用花括号包起来也是可以的。上面那种只是看起来会统一一点。看你个人喜好
	v2 := router.Group("/v2")
	v2.POST("/login", loginEndpoint)
	v2.POST("/submit", submitEndpoint)
	v2.POST("/read", readEndpoint)

	router.Run(":8080")
}

// 省略的代码 ...
```

## 中间件 (Middleware)

golang 的 net/http 设计的一大特点就是特别容易构建中间件。需要注意的是中间件只对注册过的路由函数起作用。对于分组路由，嵌套使用中间件，可以限定中间件的作用范围。中间件分为全局中间件，单个路由中间件和群组中间件。 

现代化的 Web 编程，中间件已经是必不可少的了。我们可以通过中间件的方式，验证 Auth 和身份鉴别，集中处理返回的数据等等。Gin 提供了 Middleware 的功能，并与路由紧紧相连。

### 单个路由中间件

单个路由使用中间件，只需要在注册路由的时候指定要执行的中间件即可。

```go
// 省略的代码 ...

func main() {
	router := gin.Default()

	// 注册一个路由，使用了 middleware1，middleware2 两个中间件
	router.GET("/someGet", middleware1, middleware2, handler)
  
	// 默认绑定 :8080
	router.Run()
}

func handler(c *gin.Context) {
	log.Println("exec handler")
}

// 省略的代码 ...
```

### 执行流程控制

用上面的实例代码，我们来看一下中间件是怎么执行的。

```go
// 省略的代码 ...

func middleware1(c *gin.Context) {
	log.Println("exec middleware1")
  
	//你可以写一些逻辑代码
  
	// 执行该中间件之后的逻辑
	c.Next()
}

// 省略的代码 ...
```

可以看出，中间件的写法和路由的 Handler 几乎是一样的，只是多调用 `c.Next()`。

正是有个`c.Next()`，我们可以在中间件中控制调用逻辑的变化，看下面的 `middleware2` 代码。

```go
// 省略的代码 ...

func middleware2(c *gin.Context) {
	log.Println("arrive at middleware2")
	// 执行该中间件之前，先跳到流程的下一个方法
	c.Next()
	// 流程中的其他逻辑已经执行完了
	log.Println("exec middleware2")
  
	//你可以写一些逻辑代码
}

// 省略的代码 ...
```

在 `middleware2`中，执行到 `c.Next()`时，Gin 会直接跳到流程的下一个方法中，等到这个方法执行完后，才会回来接着执行 `middleware2` 剩下的代码。

所以请求上面注册的路由 url `/someGet` ，请求先到达`middleware1`，然后到达 `middleware2`，但此时 `middleware2`调用了 `c.Next()`，所以 `middleware2`的代码并没有执行，而是跳到了 `handler`，等 `handler`执行完成后，跳回到 `middleware2`，执行 `middleware2`剩下的代码。

所以我们可以在控制台上看到以下日志输出:

```bash
exec middleware1
arrive at middleware2
exec handler
exec middleware2
```

### 路由组使用中间件

路由组使用中间件和单个路由类似，只不过是要把中间件放到 `Group` 上.

```go
// 省略的代码 ...

func main() {
	router := gin.Default()

	// 定义一个组前缀, 并使用 middleware1 中间件
  	// 访问 /v2/login 就会执行 middleware1 函数
	v2 := router.Group("/v2", middleware1)
	v2.POST("/login", loginEndpoint)
	v2.POST("/submit", submitEndpoint)
	v2.POST("/read", readEndpoint)

	router.Run(":8080")
}

// 省略的代码 ...
```