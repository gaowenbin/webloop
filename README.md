# WebLoop

Headless WebKit with a Go API. Render static HTML versions of dynamic JavaScript applications, automate browsing, run arbitrary JavaScript in a browser window context, etc., all from Go or the command line. Inspired by [PhantomJS](http://phantomjs.org/).

* [Documentation on Sourcegraph](https://sourcegraph.com/github.com/sourcegraph/webloop/tree)

[![status](https://sourcegraph.com/api/repos/github.com/sourcegraph/webloop/badges/status.png)](https://sourcegraph.com/github.com/sourcegraph/webloop)
[![xrefs](https://sourcegraph.com/api/repos/github.com/sourcegraph/webloop/badges/xrefs.png)](https://sourcegraph.com/github.com/sourcegraph/webloop)
[![funcs](https://sourcegraph.com/api/repos/github.com/sourcegraph/webloop/badges/funcs.png)](https://sourcegraph.com/github.com/sourcegraph/webloop)
[![top func](https://sourcegraph.com/api/repos/github.com/sourcegraph/webloop/badges/top-func.png)](https://sourcegraph.com/github.com/sourcegraph/webloop)
[![library users](https://sourcegraph.com/api/repos/github.com/sourcegraph/webloop/badges/library-users.png)](https://sourcegraph.com/github.com/sourcegraph/webloop)

![Screenshot of a dynamic AngularJS application running side-by-side with a statically rendered HTML equivalent, using WebLoop](https://s3-us-west-2.amazonaws.com/sourcegraph-assets/webloop.png)

## Requirements

* [Go](http://golang.org) >= 1.2rc1 (due to [#3250](https://code.google.com/p/go/issues/detail?id=3250))
* [WebKitGTK+](http://webkitgtk.org/) >= 2.0.0
* [go-webkit2](https://sourcegraph.com/github.com/sourcegraph/go-webkit2/readme)

For instructions on installing these dependencies, see the [go-webkit2
README](https://sourcegraph.com/github.com/sourcegraph/go-webkit2/readme).

To install WebLoop, run: `go get github.com/sourcegraph/webloop/...`


## Usage


### Static HTML rendering reverse proxy

The included command `static-reverse-proxy` proxies a dynamic JavaScript application and serves an equivalent statically rendered HTML website to clients. Run it with:

```
$ go install github.com/sourcegraph/webloop/...
$ static-reverse-proxy
```

For example, to proxy a dynamic application at http://example.com and serve an
equivalent statically rendered HTML website on http://localhost:13000, run:

```
$ static-reverse-proxy -target=http://example.com -bind=:13000
```

Run with `-h` to see more information.


### Rendering static HTML from a dynamic, single-page [AngularJS](http://angularjs.org) app

`StaticRenderer` is an HTTP handler that serves a static HTML version of a
dynamic web application. Use it like:

```go
staticHandler := &webloop.StaticRenderer{
        TargetBaseURL:         "http://dynamic-app.example.com",
        WaitTimeout:           time.Second * 3,
        ReturnUnfinishedPages: true
}
http.Handle("/", staticHandler)
```

See the `examples/angular-static-seo/` directory for example code. Run the included binary with:

```
$ go run examples/angular-static-seo/server.go
```

Instructions will be printed for accessing the 2 local demo HTTP servers. Run
with `-h` to see more information.


### Operating a headless WebKit and running arbitrary JavaScript in the page

```go
package webloop_test

import (
	"fmt"
	"github.com/sourcegraph/webloop"
	"os"
)

func Example() {
	ctx := webloop.New()
	view := ctx.NewView()
	defer view.Close()
	view.Open("http://google.com")
	err := view.Wait()
	if err != nil {
		fmt.Fprintf(os.Stderr, "Failed to load URL: %s", err)
		os.Exit(1)
	}
	res, err := view.EvaluateJavaScript("document.title")
	if err != nil {
		fmt.Fprintf(os.Stderr, "Failed to run JavaScript: %s", err)
		os.Exit(1)
	}
	fmt.Printf("JavaScript returned: %q\n", res)
	// output:
	// JavaScript returned: "Google"
}
```

See `webloop_test.go` for more examples.


## TODO

* [Set up CI testing.](https://github.com/sourcegraph/webloop/issues/1) This
  is difficult because all of the popular CI services run older versions of
  Ubuntu that make it difficult to install WebKitGTK+ >= 2.0.0.
* Add the ability for JavaScript code to send messages to WebLoop, similar to
  [PhantomJS's callPhantom]
  (https://github.com/ariya/phantomjs/wiki/API-Reference-WebPage#oncallback)
  mechanism.


## Contributors

See the AUTHORS file for a list of contributors.

Submit contributions via GitHub pull request. Patches should include tests and
should pass [golint](https://github.com/golang/lint).
