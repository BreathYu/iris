### client-side

- 例子

```
// 这个例子向你展示如何使用 `WriteWithExpiration`
// 根据这个"modtime"（修改时间），如果它比请求头的时间小，那么它将刷新内容
// 否则将使用客户端（99.9%的浏览器）的缓存处理机制，
// 它比iris.cache快，
// 因为服务器端没有做任何事以及不需要将响应存储在内存。
package main

import (
	"time"

	"github.com/kataras/iris"
)

const refreshEvery = 10 * time.Second

func main() {
	app := iris.New()
	app.Use(iris.Cache304(refreshEvery))
	// 上面的写法类似于下面的写法:
	// app.Use(func(ctx iris.Context) {
	// 	now := time.Now()
	// 	if modified, err := ctx.CheckIfModifiedSince(now.Add(-refresh)); !modified && err == nil {
	// 		ctx.WriteNotModified()
	// 		return
	// 	}

	// 	ctx.SetLastModified(now)

	// 	ctx.Next()
	// })

	app.Get("/", greet)
	app.Run(iris.Addr(":8080"))
}

func greet(ctx iris.Context) {
	ctx.Header("X-Custom", "my  custom header")
	ctx.Writef("Hello World! %s", time.Now())
}

```