## Go-Gin

- [安裝](#安裝)
- [一般目錄架構](#一般目錄架構)
- [基本樣板](#基本樣板)
- [中間件](#中間件)
- [Websocket](#websocket)

<br/>

### 安裝

```sh
go get -u github.com/gin-gonic/gin

import "github.com/gin-gonic/gin"
```

<br/>

### 一般目錄架構

```bash
gin-app/
│
├── internal/
│   ├── api/                # api 路由
│   │   ├── v1/             # api 版本
│   │   └── v...
│   ├── models/             # 資料庫／API 交互資料模型
│   ├── session/            # 驗證相關
│   ├── database/           # 資料庫存取
│   ├── cache/              # 快取存取
│   ├── services/           # 業務邏輯
│   ├── templates/          # html 樣板
│   └── middleware/         # 中間件
│       ├── cors.go         # 跨域授權
│       └── verify.go       # 驗證用檢查
│
├── pkg/
│   ├── logger/             # 自定義日誌格式器
│   └── utils/              # 工具函數
│
├── config/                 # 應用配置
├── logs/                   # 日誌
├── docs/                   # API 文檔 e.g. swagger
├── main.go
├── go.mod
└── go.sum
```

<br/>

### 基本樣板

```go
package main

import (
	v1 "myapp/api/v1"
	"myapp/logger"
	"myapp/data"
	"os"
	"os/signal"

	"myapp/middleware"

	"github.com/gin-gonic/gin"
	swaggerFiles "github.com/swaggo/files"
	ginSwagger "github.com/swaggo/gin-swagger" // gin-swagger middleware
)

// @title myapp
// @version 1.0
// @description Example app
// @contact.name 0xmmq
// @contact.url https://github.com/mmqnym
// @contact.email mail@mmq.dev
//
// @license.name MIT
// @license.url https://opensource.org/license/mit
// @host localhost:12345
// @BasePath /api/v1
// @Scheme http
func main() {
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	go shutdownSignal(c)

	gin.SetMode(gin.DebugMode)
	server := gin.Default()
	server.UseH2C = true
	server.Use(middleware.Cors())
	server.LoadHTMLGlob("./internal/templates/*")
	regist(server)

	logger.Sugar.Infof("Server is running on port %s", data.Config.Server.Port)
	server.Run(data.Config.Server.Port)
}

func regist(server *gin.Engine) {
	registV1(server)
	server.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
}

func registV1(server *gin.Engine) {
	group := server.Group("/api/v1")
	group.POST("/regist", v1.Regist)
	group.GET("/user", v1.GetUser)
}

func shutdownSignal(sig chan os.Signal) {
	<-sig

	logger.Sugar.Info("Server is shutting down...")
	logger.Logger.Sync()
	os.Exit(0)
}

```

<br/>

### 中間件

可以使用 `Use()` 套用在任一 group 或著僅提供給特定路由。

<br/>

```go
server := gin.Default()
server.Use(middleware.Cors()) // 套用在所有路由上

internalGroup := server.Group("/internal")
server.Use(middleware.Verify()) // 套用在 /internal/* 路由上
```

```go
group.POST("/regist", middleware.Cors(), v1.Regist) // 僅套用在 /regist 路由上
```

<br/>

- #### 常用中間件

  - CORS

  ```go
  package middleware

  import (
      "myapp/logger"

      "github.com/gin-gonic/gin"
  )

  func Cors() gin.HandlerFunc {
      return func(ctx *gin.Context) {
          origin := ctx.Request.Header.Get("Origin")
          allowedOrigins := map[string]bool{
              "https://myapp.dev":     true,
              "https://www.myapp.dev": true,
          }

          if _, ok := allowedOrigins[origin]; ok {
              ctx.Writer.Header().Set("Access-Control-Allow-Origin", origin)
          }

          ctx.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS")
          ctx.Header(
              "Access-Control-Allow-Headers",
              `User-Agent, Authorization, Content-Length, X-CSRF-Token, Origin, X-Requested-With, Content-Type, Accept,
              Cache-Control`,
          )
          ctx.Header("Access-Control-Max-Age", "172800")
          ctx.Header("Access-Control-Allow-Credentials", "true")

          defer func() {
              if err := recover(); err != nil {
                  logger.Sugar.Error(err.(error).Error())
              }
          }()

          if ctx.Request.Method == "OPTIONS" {
              ctx.AbortWithStatus(204)
              return
          }

          ctx.Next()
      }
  }
  ```

  <br/>
  - Verify（驗證 API 權限）

  ```go
  package middleware

  import (
      "myapp/logger"
      "myapp/config"

      "github.com/gin-gonic/gin"
  )

  func Verify() gin.HandlerFunc {
      return func(ctx *gin.Context) {
          accessToken := ctx.Request.Header.Get("X-Access-Token")

          if accessToken != config.AccessToken {
            ctx.AbortWithStatus(404) // 根據喜好響應
            return
          }

          ctx.Next()
      }
  }
  ```

<br/>

### Websocket

- 一般範例

```go
var upgrader = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
}

func HandleWS(ctx *gin.Context) {
	conn, err := upgrader.Upgrade(ctx.Writer, ctx.Request, nil)

	if err != nil {
		ctx.JSON(http.StatusBadRequest, gin.H{"error": "WSUpgradeError"})
		return
	}

    go handleConn(conn, alien, forceClose)
}

func handleConn(conn *websocket.Conn) {
	var userDisconnect = make(chan bool)

	defer func() {
		conn.Close()
	}()

	conn.WriteJSON(gin.H{
		"msg": "connected"
	})

    // 非阻塞讀取方式，用於監測用戶斷線
	// go func() {
	// 	_, _, err := conn.ReadMessage()

	// 	if err != nil {
	// 		// User disconnected
	// 		userDisconnect <- true
	// 	}
	// }()

    // <-userDisconnect

    // 一般訊息來往

    for {
        msgType, msg, err := conn.ReadMessage()

        if err != nil {
            println("the user disconnected")
        }

        err = ws.WriteJSON(gin.H{
		    "msg": "echo:" + string(msg),
	    })

		if err != nil {
			panic(err)
		}
    }
}
```
