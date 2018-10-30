---
layout: post
title: golang - webFramework Echo
published: true
categories: [Golang]
tags: go Echo webFramework
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
### 개요
- 웹 프레임워크.
- 경량 고속.
- 공식 페이지 https://labstack.com/echo
- 설치   go get github.com/labstack/echo
- 버전 2부터는 fasthttp 모듈을 선택해서 사용 가능하다.
- 성능 테스트 예. http://yamamaijp.hatenablog.com/entry/2016/03/10/161831
  

### 간단 예제 코드
main.go  
```
package main

import (
    "github.com/labstack/echo"
    "github.com/labstack/echo/engine/standard"
    "github.com/labstack/echo/middleware"

    "./handler"
)

func main() {
    // Echo의 인스턴스를 만든다
    e := echo.New()

    // 모든 요청에 들어가는 미들웨어(로그 등)은 여기에
    e.Use(middleware.Logger())
    e.Use(middleware.Recover())

    // 라우팅
    e.Get("/hello", handle.MainPage())

    // 서버 시작
    e.Run(standard.New(":1323"))    
}
```
handler\handle.go
```
package handler

import (
    "net/http"
    "github.com/labstack/echo"
)

func MainPage() echo.HandlerFunc {
    return func(c echo.Context) error {     
        return c.String(http.StatusOK, "Hello World")
    }
}
```
  

### Path parameter
main.go  
```
e.Get("/hello/:username", handle.MainPage())    //세미콜론이 플레이스홀더가 된다
```
handle.go
```
func MainPage() echo.HandlerFunc {
    return func(c echo.Context) error {
        username := c.Param("username")    //플레이스홀더 username의 값 추출
        return c.String(http.StatusOK, "Hello World " + username)
    }
}
```

- c.Param("username")을 c.P(0) 같이 인덱스로 지정할 수도 있다.  


### 인증
#### basic 인증
main.go  
```
e.Use(interceptor.BasicAuth())
```
auth.go
```
package interceptor

import (
    "github.com/labstack/echo/middleware"
    "github.com/labstack/echo"
)

func BasicAuth() echo.MiddlewareFunc {
    return middleware.BasicAuth(func(username, password string) bool {
        return username == "validUser" && password == "validPassword"
    })
}
```
  

#### 개별 인증
main.go  
```
e.Get("/user/apps/:username", handle.MainPage(), interceptor.BasicAuth())
```
  

### JSON
handle.go  
```
jsonMap := map[string]string{
            "foo": "bar",
            "hoge": "fuga",
        }

        return c.JSON(http.StatusOK, jsonMap)
```
  
  
```
func hello(c *echo.Context) error {
    var content struct {
        Response  string `json:"response"`
        Timestamp string `json:"timestamp"`
    }
    content.Response = "Hello, World!"
    content.Timestamp = getNow()
    return c.JSON(http.StatusOK, &content)
}
```
  

### CORS
main.go  
```
e.Use(standard.WrapMiddleware(cors.New(cors.Options{
        AllowedOrigins: []string{"http://example.com"},
    }).Handler))
```
  

### 예제 코드 해설
- 글. https://eurie.co.jp/blog/engineering/2015/12/go-lang-web-framework-echo
- 예제 코드 위치. https://github.com/eurie-inc/echo-sample
  

#### Routing과 미들웨어 이용
  
```
func Init() *echo.Echo {
    e := echo.New()

    e.Debug()

    // Set MiddleWare
    e.Use(echoMw.Logger())
    e.Use(echoMw.Recover())
    e.Use(echoMw.Gzip())

    // Set Custom MiddleWare
    e.Use(myMw.TxMiddleware(db.Init()))

    // Routes
    v1 := e.Group("/api/v1")
    {
        v1.Post("/members", api.CreateMember)
        v1.Get("/members", api.GetMembers)
        v1.Get("/members/:id", api.GetMember)
    }
    return e
}
```  
