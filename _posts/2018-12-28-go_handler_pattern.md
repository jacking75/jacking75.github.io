---
layout: post
title: golang - 핸들러 패턴
published: true
categories: [Golang]
tags: go 패턴 pattern
---

```  
type SpecialFunc func(int) bool
```  
  
SpecialFunc은 int를 받고 bool을 반환하는 함수를 뜻하는 타입이 된다.  
이것에 대해서 메소드를 생성하는 것이 가능핟.  
```
func (sf SpecialFunc) SpecialMethod() {
    ...
}
```  

  
## 일회용 구조체에 의한 핸들러 패턴  
핸들링 처리를 함수로서 넘기고, 이것을 내포한 핸들러를 반환하는 구현을 생각한다.
아마 아래와 같은 구현이 될 것이다.   
예：로그 핸들러. Record는 기록하는 로그를 나타내는 가공의 타입이다）  
  
```  
type LogHandler interface {
    Log(r *Record) error
}

type handlerFunc func(*Record) error

type FuncLogHandler struct {
    fn handlerFunc
}

func (fh *FuncLogHandler) Log(r *Record) error {
    return fh.fn(r)
}

func NewFuncLogHandler(f func(*Record) error) Handler {
    return &FuncLogHandler{fn: handlerFunc(f)}
}
```  
  
넘겨진 함수를 Log 메소드로 사용하기 위해서 일회용 구조체를 정의한다.  
함수의 메소드를 이용하면 간단하게 쓸 수 있다.  
  

  
## 함수의 메소드를 사용한 핸들러 패턴  
  
```  
type LogHandler interface {
    Log(r *Record) error
}

type handlerFunc func(*Record) error

func FuncLogHandler(fn func(r *Record) error) Handler {
    return handlerFunc(fn)
}

func (hf HandlerFunc) Log(r *Record) error {
    return hf(r)
}
``` 
  
일회용 구조체를 사용하지 않고 간단하게 쓸 수 있었다.  
  