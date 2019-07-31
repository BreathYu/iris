### simple

利用浏览器磁盘缓存的简单例子

- 例子

```
package main

import (
	"time"

	"github.com/kataras/iris"

	"github.com/kataras/iris/cache"
)
// 下面是声明了一个byte数组，里面存的是一篇markdown格式的文章，这里不做翻译
var markdownContents = []byte(`## Hello Markdown

This is a sample of Markdown contents

 

Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Routine safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.

	[this is a link](https://github.com/kataras/iris) `)


// 不应在包含动态数据的处理程序上使用缓存。
// 缓存是静态内容的一个很好的必备功能，例如“关于页面”或整个博客网站。
func main() {
	app := iris.New()
	app.Logger().SetLevel("debug")
	app.Get("/", cache.Handler(10*time.Second), writeMarkdown)
	// 在第一次请求时保存其内容，并为其提供服务，而不是去重新计算内容。
	// 10秒后，它将被清除并重置。

	app.Run(iris.Addr(":8080"))
}

func writeMarkdown(ctx iris.Context) {
	// 多次点击浏览器的“刷新”按钮
	// 您将每10秒只看到一次打印。
	println("Handler executed. Content refreshed.")

	ctx.Markdown(markdownContents)
}

/*
请注意，“HandleDir”在默认情况下使用浏览器的磁盘缓存
因此，在任何HandleDir调用之后都注册缓存处理程序，
对于更快的解决方案，服务器不需要跟踪响应
转到 https://github.com/kataras/iris/blob/master/_examples/cache/client-side/main.go
 */

```