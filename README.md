# gin-template

[![GoDoc](https://godoc.org/github.com/foolin/gin-template?status.png)](https://godoc.org/github.com/foolin/gin-template)

golang template for gin framework!


# Feature
* Easy and simple to use for gin framework.
* Use golang html/template syntax.
* Support configure master layout file.
* Support configure template file extension.
* Support configure templates directory.
* Support configure cache template.
* Support include file.
* Support dynamic reload template(disable cache mode).
* Support multiple templates for fontend and backend.

# Docs
See https://www.godoc.org/github.com/foolin/gin-template

# Install
```bash
go get github.com/foolin/gin-template
```

# Usage
```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/foolin/gin-template"
	"net/http"
)

func main() {
	router := gin.Default()

	//new template engine
	router.HTMLRender = gintemplate.Default()

	router.GET("/page", func(ctx *gin.Context) {
		ctx.HTML(http.StatusOK, "page.html", gin.H{"title": "Page file title!!"})
	})

	router.Run(":9090")
}
```

# Configure

```go
    TemplateConfig{
		Root:      "views", //template root path
		Extension: ".tpl", //file extension
		Master:    "layouts/master", //master layout file
		Partials:  []string{"partials/head"}, //partial files
		Funcs: template.FuncMap{
			"sub": func(a, b int) int {
				return a - b
			},
			// more funcs
		},
		DisableCache: false, //if disable cache, auto reload template file for debug.
	}
```


# Render

### Render with master
```go
//use name without extension
c.HTML(http.StatusOK, "index", gin.H{})
```

### Render only file(not use master layout)
```go
//use full name with extension
c.HTML(http.StatusOK, "page.html", gin.H{})
```


# Include syntax
```html
<div>
{{include "layouts/footer"}}
</div>
```

# Examples

### Basic example
```go

package main

import (
	"github.com/gin-gonic/gin"
	"github.com/foolin/gin-template"
	"net/http"
)

func main() {
	router := gin.Default()

	//new template engine
	router.HTMLRender = gintemplate.Default()

	router.GET("/", func(ctx *gin.Context) {
		//render with master
		ctx.HTML(http.StatusOK, "index", gin.H{
			"title": "Index title!",
			"add": func(a int, b int) int {
				return a + b
			},
		})
	})


	router.GET("/page", func(ctx *gin.Context) {
		//render only file, must full name with extension
		ctx.HTML(http.StatusOK, "page.html", gin.H{"title": "Page file title!!"})
	})

	router.Run(":9090")
}

```

Project structure:
```go
|-- app/views/
    |--- index.html          
    |--- page.html
    |-- layouts/
        |--- footer.html
        |--- master.html
    

See in "examples/basic" folder
```

[Basic example](https://github.com/foolin/gin-template/tree/master/examples/basic)

### Advance example
```go

package main

import (
	"github.com/gin-gonic/gin"
	"github.com/foolin/gin-template"
	"net/http"
	"html/template"
	"time"
)

func main() {
	router := gin.Default()

	//new template engine
	router.HTMLRender = gintemplate.New(gintemplate.TemplateConfig{
		Root:      "views",
		Extension: ".tpl",
		Master:    "layouts/master",
		Partials:  []string{"partials/ad"},
		Funcs: template.FuncMap{
			"sub": func(a, b int) int {
				return a - b
			},
			"copy": func() string{
				return time.Now().Format("2006")
			},
		},
		DisableCache: true,
	})

	router.GET("/", func(ctx *gin.Context) {
		//render with master
		ctx.HTML(http.StatusOK, "index", gin.H{
			"title": "Index title!",
			"add": func(a int, b int) int {
				return a + b
			},
		})
	})

	router.GET("/page", func(ctx *gin.Context) {
		//render only file, must full name with extension
		ctx.HTML(http.StatusOK, "page.tpl", gin.H{"title": "Page file title!!"})
	})

	router.Run(":9090")
}

```

Project structure:
```go
|-- app/views/
    |--- index.tpl          
    |--- page.tpl
    |-- layouts/
        |--- footer.tpl
        |--- head.tpl
        |--- master.tpl
    |-- partials/
        |--- ad.tpl
    

See in "examples/advance" folder
```

[Advance example](https://github.com/foolin/gin-template/tree/master/examples/advance)

### Multiple example
```go

package main

import (
	"html/template"
	"net/http"
	"time"

	"github.com/foolin/gin-template"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	//new template engine
	router.HTMLRender = gintemplate.New(gintemplate.TemplateConfig{
		Root:      "views/fontend",
		Extension: ".html",
		Master:    "layouts/master",
		Partials:  []string{"partials/ad"},
		Funcs: template.FuncMap{
			"copy": func() string {
				return time.Now().Format("2006")
			},
		},
		DisableCache: true,
	})

	router.GET("/", func(ctx *gin.Context) {
		// `HTML()` is a helper func to deal with multiple TemplateEngine's.
		// It detects the suitable TemplateEngine for each path automatically.
		gintemplate.HTML(ctx, http.StatusOK, "index", gin.H{
			"title": "Fontend title!",
		})
	})

	//=========== Backend ===========//

	//new middleware
	mw := gintemplate.NewMiddleware(gintemplate.TemplateConfig{
		Root:      "views/backend",
		Extension: ".html",
		Master:    "layouts/master",
		Partials:  []string{},
		Funcs: template.FuncMap{
			"copy": func() string {
				return time.Now().Format("2006")
			},
		},
		DisableCache: true,
	})

	// You should use helper func `Middleware()` to set the supplied
	// TemplateEngine and make `HTML()` work validly.
	backendGroup := router.Group("/admin", mw)

	backendGroup.GET("/", func(ctx *gin.Context) {
		// With the middleware, `HTML()` can detect the valid TemplateEngine.
		gintemplate.HTML(ctx, http.StatusOK, "index", gin.H{
			"title": "Backend title!",
		})
	})

	router.Run(":9090")
}


```

Project structure:
```go
|-- app/views/
    |-- fontend/
        |--- index.html
        |-- layouts/
            |--- footer.html
            |--- head.html
            |--- master.html
        |-- partials/
        |--- ad.html
    |-- backend/
        |--- index.html
        |-- layouts/
            |--- footer.html
            |--- head.html
            |--- master.html
        
See in "examples/multiple" folder
```

[Multiple example](https://github.com/foolin/gin-template/tree/master/examples/multiple)
