### basic

- 例子

```
package main

import "github.com/kataras/iris"

func newApp() *iris.Application {
	app := iris.New()

	// 设置一个cookie
	app.Get("/cookies/{name}/{value}", func(ctx iris.Context) {
		name := ctx.Params().Get("name")
		value := ctx.Params().Get("value")

		ctx.SetCookieKV(name, value) // <--
		// 或者: ctx.SetCookie(&http.Cookie{...})
		//
		// 如果你想设置自定义路径：
		// ctx.SetCookieKV(name, value, iris.CookiePath("/custom/path/cookie/will/be/stored"))
		//
		// 如果你只想当前的路径可以看到cookie:
		// （注意，如果服务器发送空cookie的路径，所有浏览器都是可以共用的，那么客户端应该对此负责。）
		// ctx.SetCookieKV(name, value, iris.CookieCleanPath /* 或者： iris.CookiePath("") */)
		// 更多:
		//                              iris.CookieExpires(time.Duration)
		//                              iris.CookieHTTPOnly(false)

		ctx.Writef("cookie added: %s = %s", name, value)
	})

	// 获取一个cookie
	app.Get("/cookies/{name}", func(ctx iris.Context) {
		name := ctx.Params().Get("name")

		value := ctx.GetCookie(name) // <--
		// 如果你想获取整个cookie的属性：
		// cookie, err := ctx.Request().Cookie(name)
		// if err != nil {
		//  handle error.
		// }

		ctx.WriteString(value)
	})

	// 删除一个cookie
	app.Delete("/cookies/{name}", func(ctx iris.Context) {
		name := ctx.Params().Get("name")

		ctx.RemoveCookie(name) // <--
		// 如果你想设置自定义路径：
		// ctx.SetCookieKV(name, value, iris.CookiePath("/custom/path/cookie/will/be/stored"))

		ctx.Writef("cookie %s removed", name)
	})

	return app
}

func main() {
	app := newApp()

	// GET:    http://localhost:8080/cookies/my_name/my_value
	// GET:    http://localhost:8080/cookies/my_name
	// DELETE: http://localhost:8080/cookies/my_name
	app.Run(iris.Addr(":8080"))
}

```

- 测试

```
package main

import (
	"fmt"
	"testing"

	"github.com/kataras/iris/httptest"
)

func TestCookiesBasic(t *testing.T) {
	app := newApp()
	e := httptest.New(t, app, httptest.URL("http://example.com"))

	cookieName, cookieValue := "my_cookie_name", "my_cookie_value"

	// 测试 设置一个cookie
	t1 := e.GET(fmt.Sprintf("/cookies/%s/%s", cookieName, cookieValue)).Expect().Status(httptest.StatusOK)
	t1.Cookie(cookieName).Value().Equal(cookieValue) // 验证cookie的存在，它现在应该存在。
	t1.Body().Contains(cookieValue)

	// 测试 获取一个cookie
	t2 := e.GET(fmt.Sprintf("/cookies/%s", cookieName)).Expect().Status(httptest.StatusOK)
	t2.Body().Equal(cookieValue)

	// 测试 删除一个cookie
	t3 := e.DELETE(fmt.Sprintf("/cookies/%s", cookieName)).Expect().Status(httptest.StatusOK)
	t3.Body().Contains(cookieName)

	t4 := e.GET(fmt.Sprintf("/cookies/%s", cookieName)).Expect().Status(httptest.StatusOK)
	t4.Cookies().Empty()
	t4.Body().Empty()
}

```