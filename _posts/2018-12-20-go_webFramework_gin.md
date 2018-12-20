---
layout: post
title: golang - WebFramework gin
published: true
categories: [Golang]
tags: go web gin
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  

### 개요
- [공식페이지](http://gin-gonic.github.io/gin/)
- 비교적 경량, 간단한 인터페이스를 갖추 프레임워크
- [벤치마크](https://www.techempower.com/benchmarks/)


### 설치

```
go get github.com/gin-gonic/gin
```
  

  
### hello World

```
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    // 1. http://localhost:8080/ 에 접근하면 「Hello world」라고 표시한다.
    r.GET("/", func(c *gin.Context) {
        c.String(200, "Hello world")
    })

    //2. http://localhost:8080/hoge 에 접근하면 「fuga」 라고 표시한다.
    r.GET("/hoge", func(c *gin.Context) {
        c.String(200, "fuga")
    })
    r.Run(":8080")
}
```
```
package main

import (
    "os"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    r.GET("/", func(c *gin.Context) {
        c.String(200, "hello")
    })

    port := os.Getenv("PORT")
    if len(port) == 0 {
        port = "3000"
    }
    r.Run(":" + port)
}
```
  
  

### 샘플 코드

```
package controllers

import (
    "bookshelf/models"
)

// User is
type User struct {
}

// NewUser ...
func NewUser() User {
    return User{}
}

// Get ...
func (c User) Get(n int) interface{} {
    repo := models.NewUserRepository()
    user := repo.GetByID(n)
    return user
}
```
```
package main

import (
    "bookshelf/controllers"
    "github.com/gin-gonic/gin"
    "reflect"
    "strconv"
)

func main() {
    router := gin.Default()
    router.GET("/:id", func(c *gin.Context) {
        // Pram을 처리한다
        n := c.Param("id")
        id, err := strconv.Atoi(n)
        if err != nil {
            c.JSON(400, err)
            return
        }
        if id <= 0 {
            c.JSON(400, gin.H{"status": "id should be bigger than 0"})
            return
        }
        // 데이터를 처리한다
        ctrl := controllers.NewUser()
        result := ctrl.Get(id)
        if result == nil || reflect.ValueOf(result).IsNil() {
            c.JSON(404, gin.H{})
            return
        }

        c.JSON(200, result)
    })
    router.Run(":8080")
}
```  

http://localhost:8080/[user_id]로 지정한 user_id 유저 정보를 얻는다.  

- [api made by gin framework](https://github.com/takasing/golang-gin-api)

  
  
### Gin의 Middleware 와 HandlerFunc 에서 데이터 주고 받기
- gin.\*Context.Set 과 gin.\*Context.Get 으로 동일 리퀘스트에 한해서 데이터를 주고 받는 것이 가능.
- 유저 인증으로 사용하는 예
    - Middleware 에서 유저를 인증하고, HandlerFunc에서 인증된 유저를 취득.

```
package main
import  "github.com/gin-gonic/gin"

// Midleware
func AuthRequired() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Some authorization in Authorization
        user := Authorization()
        c.Set("AuthorizedUser", user)
    }
}

// HandlerFunc
func GetProfile(c *gin.Context) {
    user := c.Get("AuthorizedUser")
}


func main() {
    r := gin.Default()
    r.Use(AuthRequired())
    r.GET("/profile", GetProfile)
    r.run()
}
```
  
  

### 미들웨어 바인딩 사용

```
package main

import (
  "github.com/gin-gonic/gin"
  "github.com/gin-gonic/gin/binding"
)

type ContactForm struct {
    Name    string `form:"name" binding:"required"`
    Message string `form:"message" binding:"required"`
}

type ContactJSON struct {
    Name    string `json:"name" binding:"required"`
    Message string `json:"message" binding:"required"`
}

func main() {
    router := gin.Default()
    router.POST("/", func(c *gin.Context) {
        var form ContactForm
        c.BindWith(&form, binding.Form)
        c.String(200, "Name: " + form.Name + "\nMessage: " + form.Message + "\n")
    })

    router.POST("/json", func(c *gin.Context) {
        var json ContactJSON
        c.BindWith(&json, binding.JSON)
        c.String(200, "Name: " + json.Name + "\nMessage: " + json.Message + "\n")
    })

    router.Run(":3000")
}
```
  
<pre>
ttp localhost:3000 name=foo message=bar
http localhost:3000/json name=foo message=bar
</pre>
  
  
  
### Binding&Validation

```
type TestForm struct {
    Name string `json:"name" binding:"required"`
    Text string `json:"text" binding:"required,max=1000"`
}
```  

- 우선 Form을 저장하는 구조체를 정의하고 태그에 Binding용 정의와 Validation용 정의를 쓴다.
- Binding 태그 안에 실시하는 Validation을 열거한다. 여러 개 있을 경우 컴마 단락으로 쓰다.
    - 평가는 왼쪽에서 차례대로, Error가 있을 경우 이후의 평가는 하지 않는다.
    - [이용할 수 있는 Validation은 이곳을 참조](http://godoc.org/gopkg.in/bluesuncorp/validator.v5#hdr-Baked_In_Validators_and_Tags)
- 아래 json 데이터를 보낸다고 가정
  
<pre>
{"name": "taro", "text": "hogehoge"}
</pre>
  
```
r.POST("/test", func(c *gin.Context) {
    var form TestForm
    c.Bind(&form)

    log.Println(form.Name) // taro
    log.Println(form.Text) // hogehoge
    ...
```  

- HandlerFunc 중에서 정의한 Form 구조체를 선언하고, gin.Context의 Bind 메서드에 포인터를 주면 좋게 Binding 해준다.
  
```
r.POST("/test", func(c *gin.Context) {
    var form TestForm
    if err := c.Bind(&form); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"status": "BadRequest"})
        return
    }

    // 이하 정상 처리
```
  
- 에러 상세 처리

```
r.POST("/test", func(c *gin.Context) {
    var form TestForm
    if err := c.Bind(&form); err != nil {
        errors := err.(*validator.StructErrors)
        log.Println("Struct:", errors.Struct)
        for k, v := range errors.Errors {
            log.Println("Key:", k)
            log.Println("Field:", v.Field)
            log.Println("Param:", v.Param)
            log.Println("Tag:", v.Tag)
            log.Println("Kind", v.Kind)
            log.Println("Type:", v.Type)
            log.Println("Value", v.Value)
            log.Println("==========")
        }
        log.Println(errors.StructErrors)
        c.JSON(http.StatusBadRequest, gin.H{"status": "BadRequest"})
        return
    }

    // 아래는 정상 처리
```
  
- [출처](http://qiita.com/emonuh/items/3b531fded9b5ede9d93a)
  
  

### 볼 글
- [go-imageupload](https://github.com/olahol/go-imageupload)
- [(일어) gom으로 gin을 run 한다](http://qiita.com/wataru420/items/a0a7cd1e218013bb03b8)
- [(일어) gb로 gin을 run 한다](http://qiita.com/wataru420/items/4533af8924c84b9b3029)
- [(일어) Gin/bindata 로 싱글 바이너리를 시험해 보기(+React)](http://qiita.com/wadahiro/items/4173788d54f028936723)
- [REST Microservices in Go with Gin](http://txt.fliglio.com/2014/07/restful-microservices-in-go-with-gin/)
    - [complete example](https://github.com/benschw/go-todo)
