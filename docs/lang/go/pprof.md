



## 代码配置



gin 使用

``` go
package httpserv

import (
	"github.com/gin-contrib/pprof"
	"github.com/gin-gonic/gin"
	"github.com/imattdu/go-standard/middleware"
)

func Start() {
	r := gin.New()
	// 使用pprof
	pprof.Register(r)
	r.Use(middleware.GinLogger(), middleware.GinRecovery(true),
		middleware.TraceLoggerMiddleware())
	Config(r)
	_ = r.Run() // 监听并在 0.0.0.0:8080 上启动服务
}

```



使用默认的 http.DefaultServeMux

```go
import _ "net/http/pprof"


http.ListenAndServe("0.0.0.0:8000", nil)
```



自定义的 server pprof中init的配置

```go
r.HandleFunc("/debug/pprof/", pprof.Index)
r.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
r.HandleFunc("/debug/pprof/profile", pprof.Profile)
r.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
r.HandleFunc("/debug/pprof/trace", pprof.Trace)
```





``` go
func init() {
	http.HandleFunc("/debug/pprof/", Index)
	http.HandleFunc("/debug/pprof/cmdline", Cmdline)
	http.HandleFunc("/debug/pprof/profile", Profile)
	http.HandleFunc("/debug/pprof/symbol", Symbol)
	http.HandleFunc("/debug/pprof/trace", Trace)
}
```









``` go
go tool pprof -http=:9000 http://localhost:8080/debug/pprof/allocs
```





``` sh
brew install graphviz
```

