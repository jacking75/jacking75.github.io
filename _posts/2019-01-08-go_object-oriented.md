---
layout: post
title: golang - OOP 프로그래밍 하기
published: true
categories: [Golang]
tags: go OOP
---
### 캡슐화
- Go에서는 구조체와 해당 필드, 함수, 메소드의 범위는 이름의 선두가 대문자, 소문자로 결정된다.
- 대문자이면 public, 소문자라면 같은 패키지 내로 닫힌 범위로.
- 그러므로 아래와 같이 쓰는 것으로 캡슐화를 실현할 수 있다.
  
```
//human.go
package class

type Human struct {
    // 필드 이름을 소문자로 하여 첫 스코프를 같은 패키지 내로 닫는다
    name string
}

// 생성자
// 구조체가 public이라도 그 필드가 닫힌 스코프의 경우
// 패키지 외부에서 {}을 사용한 초기화는 할 수 없다.
func NewHuman(name string) *Human {
    return &Human{ name: name }
}

// 캡슐화
// 메소드 이름을 대문자부터 시작함으로써 public 범위로 한다.
func (human Human) GetName() string {
    return human.name
}

// setter는 수신기를 포인터로 하지 않으면 값이 변경되지 않는다
func (human *Human) SetName(name string) {
    human.name = name
}
```
  
```
// main.go
package main

import (
    "./class"
    "fmt"
)

func main() {
    human := class.NewHuman("alice")
    fmt.Println(human.GetName())
    human.SetName("bob")
    fmt.Println(human.GetName())
}
```
  
<pre>
bash
$ go run main.go
alice
bob
</pre>  
  
  
  
#### 주의
- human.go에서 생성자라고 썼지만 Go에는 생성자라는 구문은 없다. 대신 New로 시작되는 함수를 정의하고, 그 내부 구조체를 생성하는 것이 통례.
- 같은 패키지 내에서는 액세서를 경유하지 않고 접근할 수 있다.
  
  
### Embed
- Go에 계승은 없지만 대신 다른 형을 심는(Embed) 방식으로 행동을 확장할 수 있다.
- 내장된 형(아래에서는 Human)의 필드와 메소드는 박은 형(아래에서는 Man)이 구현한 것처럼 행세한다.

```
// human.go
/*생략*/

// Human의 메소드
func (human Human) Check() {
    fmt.Printf("%s is a Human.\n", human.name)
}

type Man struct {
    *Human // Human을 내장
}

func NewMan(name string) *Man {
    return &Man{ Human: NewHuman(name) }
}
```
  
```
// main.go
/*생략*/

func main() {
    man := class.NewMan("bob")
    man.Check() // Human의 메소드를 자신이 구현한 것처럼 불러낼 수 있다.
}
```  

<pre>
// bash
$ go run main.go
bob is a Human.
</pre>  
  

  
### 오버 라이드
- 동명의 메소드를 자신의 형을 수신기로 하고 재정의함으로써 메소드를 오버 라이드 할 수 있다.

```
// human.go
/*생략*/

func (man Man) Check( ) {
    fmt.Printf("%s is a Man.\n", man.name)
}
```  

<pre>
방금 전과 같은 main.go를 실행하면
bash
$ go run main.go
bob is a Man.
</pre>  


  
#### 주의
박은 형(Man)을 내장한 형(Human)로 다룰 수 없다. 예를 들어,  
```
var man *class.Human = class.NewMan("bob")
```  

로 하면 컴파일 오류가 된다.  
이는 처음에 올린 공식 문서에도 있듯이 Go에는 타입 계층이 없어서 Human과 Man은 완전히 다른 형태로 되기 때문이다.  
Embed는 어디까지나 내장이지 계승이 아니다.  
  

  
### 폴리 모피즘
- Go에서는 승계를 사용한 폴리 모피즘은 실현 불가능하지만, interface을 구현함으로써 폴리 모피즘을 실현할 수 있다.  

```
// human.go
/*생략*/

type Speaker interface {
    Speak()
}

/*생략*/

// Human에 Speaker를 구현
func (human Human) Speak() {
    fmt.Println("I am Humman.")
}

/*생략*/

//Human은 내장하지 않는다
type Woman struct {
    name string
}

func NewWoman(name string) *Woman {
    return &Woman{ name: name }
}

// Woman에 Speaker를 구현
func (woman Woman) Speak() {
    fmt.Println("I am Woman.")
}
```
  
```
// main.go
/*생략*/

func main() {
    human := class.NewHuman("alice")
    woman := class.NewWoman("emily")

    // Human도 Woman도 Speaker 인터페이스로 다룰 수 있다.
    speak(human)
    speak(woman)
}

func speak(speaker class.Speaker) {
    speaker.Speak()
}
```  

<pre>
// bash
$ go run main.go
I am a Humman.
I am a Woman.
</pre>  
  

  
### interface를 구현된 타입을 내장
- 어떤 interface를 구현된 타입을 내장한 구조체 또한 그 interface로 다룰 수 있다.
- 위의 예에서는 Man은 Human를 내장하고 있으므로, Man자체는 Speaker를 구현하지 않아도 Speaker로서 다룰 수 있다.
  
```
// main.go
/*생략*/

func main() {
    human := class.NewHuman("alice")
    man := class.NewMan("bob")
    woman := class.NewWoman("emily")
    speak(human)
    speak(man)
    speak(woman)
}

func speak(speaker class.Speaker) {
    speaker.Speak()
}
```  

<pre>
bash
$ go run main.go
I am a Humman.
I am a Humman.
I am a Woman.
</pre>  

물론 Man이 Speak메서드를 오버 라이드 할 수도 있다.  
   

   
#### 출처
- http://qiita.com/kitoko552/items/a6698c68379a8cd8b999
