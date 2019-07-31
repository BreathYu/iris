### hello-world
一个入门案例

- 例子

```go
package main

import (
	"github.com/kataras/iris"

	"github.com/kataras/iris/middleware/logger"
	"github.com/kataras/iris/middleware/recover"
)

func main() {
	app := iris.New()
	app.Logger().SetLevel("debug")
	// 可选，添加两个内置中间件
	// recover中间件在服务出现异常时，恢复服务的正常运行
	// logger日志中间件可以将请求记录在终端
	app.Use(recover.New())
	app.Use(logger.New())

	// 方法:   GET
	// 访问地址: http://localhost:8080
	app.Handle("GET", "/", func(ctx iris.Context) {
		ctx.HTML("<h1>Welcome</h1>")
	})

	// 下面这个类似写法 app.Handle("GET", "/ping", [...])
	// 方法:   GET
	// 访问地址: http://localhost:8080/ping
	app.Get("/ping", func(ctx iris.Context) {
		ctx.WriteString("pong")
	})

	// 方法:   GET
	// 访问地址: http://localhost:8080/hello
	app.Get("/hello", func(ctx iris.Context) {
		ctx.JSON(iris.Map{"message": "Hello Iris!"})
	})

	// http://localhost:8080
	// http://localhost:8080/ping
	// http://localhost:8080/hello
	app.Run(iris.Addr(":8080"), iris.WithoutServerError(iris.ErrServerClosed))
}

```

