### securecookie

一个安全的cookie

- 例子

```
package main

// developers can use any library to add a custom cookie encoder/decoder.
// At this example we use the gorilla's securecookie package:
// $ go get github.com/gorilla/securecookie
// $ go run main.go

import (
	"github.com/kataras/iris"

	"github.com/gorilla/securecookie"
)

var (
	// AES算法仅支持key的长度为16、24或者32位
	// 您需要准确提供该key长度，或者从传递的key值中获取长度。
	hashKey  = []byte("the-big-and-secret-fash-key-here")
	blockKey = []byte("lot-secret-of-characters-big-too")
	sc       = securecookie.New(hashKey, blockKey)
)

func newApp() *iris.Application {
	app := iris.New()

	// 设置 一个cookie
	app.Get("/cookies/{name}/{value}", func(ctx iris.Context) {
		name := ctx.Params().Get("name")
		value := ctx.Params().Get("value")

		ctx.SetCookieKV(name, value, iris.CookieEncode(sc.Encode)) // <--

		ctx.Writef("cookie added: %s = %s", name, value)
	})

	// 获取 一个cookie
	app.Get("/cookies/{name}", func(ctx iris.Context) {
		name := ctx.Params().Get("name")

		value := ctx.GetCookie(name, iris.CookieDecode(sc.Decode)) // <--

		ctx.WriteString(value)
	})

	// 删除 一个cookie
	app.Delete("/cookies/{name}", func(ctx iris.Context) {
		name := ctx.Params().Get("name")

		ctx.RemoveCookie(name) // <--

		ctx.Writef("cookie %s removed", name)
	})

	return app
}

func main() {
	app := newApp()
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
	// 请注意，这将不起作用，因为它并不总是返回相同的值：
	// cookieValueEncoded, _ := sc.Encode(cookieName, cookieValue)
	t1.Cookie(cookieName).Value().NotEqual(cookieValue) // 验证cookie的存在，并且值不等于原值。
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