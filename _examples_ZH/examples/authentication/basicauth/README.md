### basicauth

基本身份信息认证

- 例子

```go
package main

import (
	"time"

	"github.com/kataras/iris"
	"github.com/kataras/iris/middleware/basicauth"
)

func newApp() *iris.Application {
	app := iris.New()

	authConfig := basicauth.Config{
		Users:   map[string]string{"myusername": "mypassword", "mySecondusername": "mySecondpassword"},
		Realm:   "Authorization Required", // 认证失败后返回提示信息
		Expires: time.Duration(30) * time.Minute,
	}

	authentication := basicauth.New(authConfig)

	// 如果要对所有路由进行权限认证，在所有路由前面使用 app.Use(authentication) （或者 在app.Run方法之前使用 app.UseGlobal(authentication) ）
	// 针对单个路由使用权限认证，栗子:
	/*
		app.Get("/mysecret", authentication, h)
	*/

	app.Get("/", func(ctx iris.Context) { ctx.Redirect("/admin") })

	// 对于路由组要加权限认证，如下

	needAuth := app.Party("/admin", authentication)
	{
		//http://localhost:8080/admin
		needAuth.Get("/", h)
		// http://localhost:8080/admin/profile
		needAuth.Get("/profile", h)

		// http://localhost:8080/admin/settings
		needAuth.Get("/settings", h)
	}

	return app
}

func main() {
	app := newApp()
	// open http://localhost:8080/admin
	app.Run(iris.Addr(":8080"))
}

func h(ctx iris.Context) {
	username, password, _ := ctx.Request().BasicAuth()
	// 第三个参数它将始终为true，
	// 因为中间件确保了这一点，否则将不会执行此处理程序。

	ctx.Writef("%s %s:%s", ctx.Path(), username, password)
}

```

- 测试

```go
package main

import (
	"testing"

	"github.com/kataras/iris/httptest"
)

func TestBasicAuth(t *testing.T) {
	app := newApp()
	e := httptest.New(t, app)

	// 这个请求会重定向到 路由/admin，请求没有带基本的身份验证信息（账号和密码）
	e.GET("/").Expect().Status(httptest.StatusUnauthorized)
	// 没有带基本的身份验证信息
	e.GET("/admin").Expect().Status(httptest.StatusUnauthorized)

	// 使用合法的身份信息（账号和密码）
	e.GET("/admin").WithBasicAuth("myusername", "mypassword").Expect().
		Status(httptest.StatusOK).Body().Equal("/admin myusername:mypassword")
	e.GET("/admin/profile").WithBasicAuth("myusername", "mypassword").Expect().
		Status(httptest.StatusOK).Body().Equal("/admin/profile myusername:mypassword")
	e.GET("/admin/settings").WithBasicAuth("myusername", "mypassword").Expect().
		Status(httptest.StatusOK).Body().Equal("/admin/settings myusername:mypassword")

	// 使用不合法的身份信息（账号和密码）
	e.GET("/admin/settings").WithBasicAuth("invalidusername", "invalidpassword").
		Expect().Status(httptest.StatusUnauthorized)
}

```