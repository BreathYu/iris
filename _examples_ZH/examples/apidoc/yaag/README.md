### yaag
api 文档生成器

- 例子

```go
package main

import (
	"github.com/kataras/iris"

	"github.com/betacraft/yaag/irisyaag"
	"github.com/betacraft/yaag/yaag"
)

/*
	go get github.com/betacraft/yaag/...
*/

type myXML struct {
	Result string `xml:"result"`
}

func main() {
	app := iris.New()

	yaag.Init(&yaag.Config{ // <- 重要！初始化这个中间件
		On:       true,
		DocTitle: "Iris",
		DocPath:  "apidoc.html",
		BaseUrls: map[string]string{"Production": "", "Staging": ""},
	})
	app.Use(irisyaag.New()) // <- 重要！注册这个中间

	app.Get("/json", func(ctx iris.Context) {
		ctx.JSON(iris.Map{"result": "Hello World!"})
	})

	app.Get("/plain", func(ctx iris.Context) {
		ctx.Text("Hello World!")
	})

	app.Get("/xml", func(ctx iris.Context) {
		ctx.XML(myXML{Result: "Hello World!"})
	})

	app.Get("/complex", func(ctx iris.Context) {
		value := ctx.URLParam("key")
		ctx.JSON(iris.Map{"value": value})
	})

	// 运行我们的服务
	// 
	// “yaag”的文档没有注明以下内容，但在Iris我们非常谨慎地对您提供了这些内容
	// 每次请求都会重新生成和更新“apidoc.html”文件
	// 重现：
	// 编写并调用这些测试程序，"apidoc.html"将会自动生成并保存这次API请求的数据
	// 在生成环境中关闭这个中间件
	//
	// 案例用法
	// 访问所有路径并打开生成的“apidoc.html”文件以查看自动生成的API文档。
	app.Run(iris.Addr(":8080"))
}
```

