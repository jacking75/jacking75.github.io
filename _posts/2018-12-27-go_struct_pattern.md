---
layout: post
title: golang - 구조체 구현 패턴
published: true
categories: [Golang]
tags: go 구조체 struct
---
[출처](http://blog.monochromegane.com/blog/2014/03/23/struct-implementaion-patterns-in-golang/)    
   
- 생성자 함수
- export에 의한 접근 허가
- 인터페이스에 의한 다형성
- 구조체에 의한 다형성
- 구조체에 의한 서브 클래스, responsibility
- 구조체에 의한 이양
- 함수에 의한 이양
  
  

## 생성자 함수
Go 언어는 구조체의 생성자가 없으므로 구조체를 초기화 하려면 구조체가 속하는 패키지에 생성자 함수를 정의한다.  
관례는 함수 이름으로 New + 구조체 이름() 이 이용되고 있다.  

```
type StructA struct {
	Name string
}

// 초기화 시에 하고 싶은 처리
func (self *StructA) SomeInitialize() {
	// Some initialize
}

// 생성자 함수를 정의
func NewStructA(name string) *StructA {
	structA := &StructA{Name: name}
	structA.SomeInitialize()
	return structA
}

func main() {
	_ = NewStructA("name")
}
```  

생성자 함수를 사용하는 것에는 다음과 같은 장점이 있다.  
- 구조체의 생성과 동시에 수행 할 작업을 이용 측에서 숨긴다.
- 매입 구조체를 이용 측에서 숨긴다(구조체의 구조를 이용 측에서 숨긴다. 매입 구조의 존재를 숨기는 것에도 사용할 수 있다)  
   

   
## export에 의한 접근 허가
Go 언어에서는 패키지 외부에 대한 접근 허가는 이름의 첫 글자를 대문자로하는 것으로 한다(내보내기).  
이것을 이용하여 싱글톤 같은 인스턴스 생성을 제어 할 수 있다.  

```
package singleton

// 구조체의 이름을 소문자로 해서 패키지 밖으로 export 하지 않는다
type singleton struct {
}

// 인스턴스를 저장하는 변수도 소문자로 하여 export 하지 않는다
var instance *singleton

// 인스턴스 취득용 함수만 export 해 두고, 여기에서 인스턴스를 접근할 수 있게 해준다.
func GetInstance() *singleton {
	if instance == nil {
	     instance = &singleton{}
	}
	return instance
}
```  

생략 서식 :=을 사용하여 구조체 자체를 export 하지 않아도 사용할 수 있다.  
```
singleton := singleton.GetInstance()
```  
  

  
## 인터페이스에 의한 다형성
Go 언어는 타입의 상속 기능을 제공하지 않지만 인터페이스를 이용하여 다형성을 구현할 수 있다.  

```
// 인터페이스 정의
type SomeBehivor interface {
	DoSomething(arg string) string
}

// 구조체 정의
type StructA struct {
}

// 인터페이스 구현
func (self *StructA) DoSomething(arg string) string {
	return arg
}

// 다형성
func main() {
	// 인터페이스 타입 변수에 저장할 수 있다
	var behivor SomeBehivor
	behivor = &StructA{}
	behivor.DoSomething("A")
}
```  
  
네트워크 라이브러리에 애플리케이션 측의 콜백 함수 전달    
출처: https://github.com/gansidui/gotcp  
```
type ConnCallback interface {
	// OnConnect is called when the connection was accepted,
	// If the return value of false is closed
	OnConnect(*Conn) bool

	// OnMessage is called when the connection receives a packet,
	// If the return value of false is closed
	OnMessage(*Conn, Packet) bool

	// OnClose is called when the connection closed
	OnClose(*Conn)
}

type Server struct {
	config    *Config         // server configuration
	callback  ConnCallback    // message callbacks in connection
	protocol  Protocol        // customize packet protocol
	exitChan  chan struct{}   // notify all goroutines to shutdown
	waitGroup *sync.WaitGroup // wait for all goroutines
}

type Callback struct{}

func (this *Callback) OnConnect(c *gotcp.Conn) bool {
	// todo
	return true
}

func (this *Callback) OnMessage(c *gotcp.Conn, p gotcp.Packet) bool {
	// todo
	return true
}

func (this *Callback) OnClose(c *gotcp.Conn) {
	// todo
}


// NewServer creates a server
func NewServer(config *Config, callback ConnCallback, protocol Protocol) *Server {
	return &Server{
		config:    config,
		callback:  callback,
		protocol:  protocol,
		exitChan:  make(chan struct{}),
		waitGroup: &sync.WaitGroup{},
	}
}


srv := gotcp.NewServer(config, &Callback{}, &echo.EchoProtocol{})
```

    
  
## 구조체에 의한 다형성
다형성을 실현하면서 구조체에 공통의 구현 및 멤버를 정의 할 수 있다.  
이 경우 인터페이스를 구현한 구조체를 준비하고, 이것을 포함하는 방법을 취한다.  

```  
// 인터페이스 정의
type SomeBehivor interface {
	DoSomething() string
}

// 공통 구조체 정의
type DefaultStruct struct {
	// 공통 멤버
	Name string
}

// 공통 구조체에 인터페이스를 구현
func (self *DefaultStruct) DoSomething() string {
	return self.Name
}

// 공통 구조체를 포함
type StructA struct {
	*DefaultStruct
}

func main() {
	// 인터페이스 타입의 변수에 저장할 수 있다
	behivor := &StructA{&DefaultStruct{"A"}}

	// 곹옹 구현과 멤버를 사용할 수 있다
	behivor.DoSomething()
}
```  

그러나 이 경우 이용 측이 초기화 시에 포함 구조를 의식 할 필요가 있기 때문에 위의 생성자 함수를 준비하는 것이 좋다.  
```
// 생성자 함수
func NewStructA(name string) *StructA {
	return &StructA{&DefaultStruct{"A"}}
}
// 이용 측은 포함 구조체의 존재를 의식하지 않는다
behivor := NewStructA("A")
```  
  

  
## 구조에 의한 서브 클래스, responsibility
Go 언어에서는 포함된 구조체가 원래 구조체의 구현을 사용할 수 없다(Dispatch back 되지 않는다고 말한다).  
따라서 TemplateMethod 패턴처럼 자식 클래스에 처리 구현을 맡기는 기능을 만드는 경우 포함된 구조체의 메서드에 대해서 레시버가 되는 원래의 구조 참조를 전달해야 한다.  

```
type Behivor interface {
	DoSomethingByOther()
}

type Abstract struct {
}

// 내부에서 원 구조체의 구현을 이용하는 메소드
func (self *Abstract) DoSomething(behivor Behivor) {
	behivor.DoSomethingByOther()

	// 여기에서 Abstract에 Behivor를 포함하는 등 메소드가 있는 상태에서 컴파일을 통과하도록 
	// self.DoSomethingByOther() 로도 panic: runtime error: invalid memory address or nil pointer dereference가
	// 발생하고、Dispatch back 되지 않는 것을 알 수 있다
}

// 위에서 정의한 구조체를 포함
type Concrete struct {
	*Abstract
}

// 인터페이스 를 구현
func (self *Concrete) DoSomethingByOther() {
	// Do something
}

func main() {
	concrete := &Concrete{}
	// 레시버가 되는 자기 자신을 넘긴다
	concrete.DoSomething(concrete)
}
```
  
이 레시버를 스스로 지정하는 방법은 상속을 사용할 수 없는 언어에서 Client-specified self 패턴라는 이름으로 사용 되고 있는 것 같다.  
그러나 구조체 간의 종속성이 높아지기 때문에 아무래도 상속 관계가 필요한 경우를 제외하고 후술하는 구조체에 의한 양도 또는 함수에 의한 이양을 검토하는 것이 좋다.  
  

  
## 구조체에 의한 이양
Go 언어에서는 처리의 구현을 적절한 책임을 가지는 구조체에 맡기는 이양은 쉽게 실현 할 수 있다.  
이것에는 구조체를 멤버로 가지고 필요에 따라 그 구조체의 메소드를 부르도록 한다.  
또한 Strategy 패턴처럼 인터페이스에 이양하도록 설계 해두면 의존성을 더 낮출 수있을 것이다.  

```
type Strategy interface {
	DoSomething()
}

type ConcreteStrategy struct {
}

func (self *ConcreteStrategy) DoSomething() {
	// Do something
}

type StructA struct {
	// 이양처를 멤버로 가진다
	strategy Strategy
}

func (self *StructA) Operation() {
	// 포함한 구조체에 처리를 이양
	self.strategy.DoSomething()
}
```
  

  
## 함수에 의한 이양
Go 언어에서는 함수를 퍼스트 클래스 타입으로 취급하기 때문에 이양하는 작업을 함수로 전달히는 방법도 취할 수 있다.  
Go 언어의 기본 패키지를 보면, 이양 처리를 외부에서 주입하는 기능을 제공하는 경우, 함수를 전달하도록 설계 되어 있는 것이 많은 듯하다.  

```
// 함수 탕비을 정의
type DoSomething func()


type StructA struct {
}

// 함수 타입을 인수로 받아들이는 메소드를 정의
func (self *StructA) Operation(strategy DoSomething) {
	// 받아들인 함수 타입에 처리를 이양
	strategy()
}

func main() {
	structA := &StructA{}

	// 함수 리터럴에서 인수로서 넘긴다
	structA.Operation(func(){
	     // Do something
	})
}
```  
  