---
layout: post
title: golang - channel 사용하기
published: true
categories: [Golang]
tags: go channel
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  

# 채널 사용하기
  
## 받기 전용, 보내기 전용 채널 선언
받기 전용  

```
c <-chan int
```
  
보내기 전용  
```
c chan<- int
```

       
## 다른 고루틴에 동기적으로 통신하고 싶은 경우
2개의 채널이 서로 동기적으로 데이터를 주고 받음  

```
reqc := make(chan chan int)

go func(reqc chan chan int) {
  for {
    select {
    case resc := <-reqc:
      log.Println("another goroutine receives a request")
      resc <- 42
    }
  }
}(reqc)

request := func() int {
  req := make(chan int)
  reqc <- req
  return <-req
}

log.Println("the main goroutine receives a response:", request())
```
  

## 채널의 데이터 비우기
출처 [How can I explicitly empty a channel?](https://stackoverflow.com/questions/26143091/how-can-i-explicitly-empty-a-channel)  
- 채널을 1개의 스레드에서만 접근

```
for len(ch) > 0 {
  <-ch
}
```  

- 채널을 복수 스레드에서 접근  

```
L:
for {
    select {
    case <-c:
    default:
       break L
    }
}
```
  

## Go언어에서 비동기 처리 결과 받기
### 반환 값이 없다
반환 값이 없어 처리가 끝나면 좋은 경우  
```
// bool로도 괜찮지만 빈 struct
done := make(chan struct{}, 0)

go func() {
    //뭔가 처리를 하다

    close(done)
}()

// chan이 close 될 때까지 차단
<-done
```

값의 교환이 없는 경우는 간단하다.
  

 #### Tips: 빈 struct에 대해서
값의 교환이 불필요한 경우의 channel에는 빈 struct가 좋다. 사이즈가 제로이기 때문.
참고:http://dave.cheney.net/2014/03/25/the-empty-struct  


#### Tips: close의 defer
실행된 goroutine 측에서 분기로 return 할 곳이 몇 개 있는 경우 등 close를 잊지 않도록 하고 싶으면 defer 한다  
```
done := make(chan struct{}, 0)

go func() {
    //함수를 나올 때 defer에서 close 호출
    defer close(done)

    //뭔가 처리를 한다
}()

<-done
```
    

### 반환 값이 1개
- int을 받을 경우  

```
intChan := make(chan int, 1)

go func() {
    //뭔가 처리를 한다

    intChan <- someValue
}()

value := <-intChan
```  

- error을 받을 경우  

```
errChan := make(chan error, 1)

go func() {
    //뭔가 처리를 한다

    errChan <- err
}()

err := <-errChan
```  

channel을 준비하고, 거기에 송수신할 뿐이다.  
  

 #### Tips: channel의 버퍼 사이즈
channel의 버퍼가 1이 되는 것은 실행된 goroutine 측에서 값을 보낼 때 차단되지 않음을 확실히 하기 위해서이다.  
버퍼 0의 경우 메인 측이 다른 처리를 하고 있어서 channel 수신을 할 때까지 goroutine 측의 channel로의 전송을 기다리게 된다.   처리가 완료된 goroutine은 적은 자원 관점에서도 빨리 처리 해야 하기 때문에, channel에 대한 송신이 차단되지 않도록 하고 있다.  
    

### 반환 값이 2개

```
intChan := make(chan int, 1)
errChan := make(chan error, 1)

go func() {
    //뭔가 처리를 하다

    if err != nil {
        errChan <- err
    } else {
        intChan <- someValue
    }
}()

select {
case value := <- intChan:
    //처리를 성공한 경우의 처리
case err := <- errChan:
    //처리를 실패한 경우의 처리
}
```  

복수의 channel을 기다릴 때는 select를 사용한다.  
이 예에서는 int와 error를 반환하고 성공한 경우와 실패한 경우로 나뉘어 있어서 비교적 간단하다.  
  

#### Note: select를 사용하지 않을 경우
또한 select가 아니라 다음과 같은 코드로 결과를 기다리는 경우, 이 예에서는 데드락 된다.  
```
intChan := make(chan int, 1)
errChan := make(chan error, 1)

go func() {
    //뭔가 처리를 하다

    if err != nil {
        errChan <- err
    } else {
        intChan <- someValue
    }
}()

value := <- intChan
err := <- errChan
if err != nil {
    //처리를 실패한 경우의 처리
} else {
    //처리를 성공한 경우의 처리
}
```  

이 예에서 goroutine 측에서는 조건 분기에 의해서 errChan 이나 intChan 양쪽에서만 값을 송신하기 때문에 양쪽 channel에서 값을 받으려고 하면 값이 없는 송신도 되지 않으므로 계속 기다리기 된다.  
  
여기에서는 대신 goroutine 측에서 양쪽 channel에 송신하는 것에서 교착 상태는 없다.    
```
go func() {
    //뭔가 처리를 한다

    errChan <- err
    intChan <- someValue
}()
```  
  
그러나 이 경우 channel의 버퍼가 1 이상이 아니면 데드락 한다.  
channel의 송수신은 블록이 발생할 가능성이 항상 있기 때문에, 블록이나 데드락을 의식하고 사용하도록 한다.  
  

### 반환 값이 3개

```
intChan := make(chan int, 1)
strChan := make(chan string, 1)
errChan := make(chan error, 1)

go func() {
    //뭔가 처리를 하다

    if err != nil {
        errChan <- err
    } else {
        intChan <- someInt
        strChan <- someStr
    }
}()

select {
case value := <- intChan:
    //성공 루트라서 또 하나의 결과도 받는다
    strChan <- someStr

    //처리를 성공한 경우의 처리

case err := <- errChan:
    //처리를 실패한 경우의 처리
}
```  
  
슬슬 읽기 어렵게 되었다.  
intChan에서 결과를 받은 후 strChan으로도 결과를 받지 않으면 안 된다.  
만약 channel의 버퍼를 제로로 하면, intChan, strChan의 받는 순서가 반대가 되어 데드락 된다.  
  
  
### 결과용 구조체를 사용
Go의 철학으로 "읽기 쉬움"은 매우 중요하므로 결과를 묶는 구조체를 써서 간단하게 해보자.  
  
구조체:
```
type Foo struct {
    Bar int
    Baz string
    Err error
}
```  
그리고:  
```
fooChan := make(chan Foo, 1)

go func() {
    //뭔가 처리를 하다

    var f Foo
    if err != nil {
        f.Err = err
    } else {
        f.Bar = someBar
        f.Baz = someBaz
    }
    fooChan <- f
}()

foo := <- fooChan
if foo.Err != nil {
    //처리가 실패한 경우의 처리
} else {
    //처리가 성공한 경우의 처리
}
```  
  
다루는 channel이 하나로 간단하게 되었다.  
반환 값이 2개 이상의 경우 이렇게 결과용 구조체로 정리하면 깔끔한 경우가 많다.  
  

#### Tips: 값 or 포인터
위 예에서는 channel의 형태는*Foo 가 아닌 Foo로 되어 있다.  
이는 멤버가 적거나 사이즈 작은 것으로 복제 비용이 작아서 값 복사로 메모리 확보가 발생하지 않기 때문이다.  
Foo의 사이즈는 20 바이트 또는 24바이트(int:4 or 8, string:8, error:8)이다. 이 정도의 복사면 메모리 확보보다 훨씬 저렴.  
함수 내에서 확보한 값의 포인터를 함수 외에 전달하는 경우 값을 스택에 둘 수 없기 때문에 힙에 메모리를 확보한다. 메모리 확보와 GC는 비용이 높기 때문에 의식하도록 한다.  
  

### Note: 반환 값이 3개 이상인 것은 좋지 않은가?
반환 값이 3개 이상인 시점에서 원래 함수의 설계가 좋지 않을 가능성이 있다.  
다음 예에서는 X좌표와 Y좌표의 두 float와 error의 3개의 반환 값을 반환 하는 것이 아니라 Point 구조체와 error의 2개의 반환 값을 반환하도록 한다.  

```
type Point struct {
    X float64
    Y float64
}
errChan := make(chan error, 1)
pointChan := make(chan Point, 1)

go func() {
    //뭔가 처리를 하다
    p, err := someFunc()
    errChan <- err
    pointChan <- p
}()

err := <- errChan
point := <- pointChan
if err != nil {
    //처리가 실패한 경우의 처리
} else {
    //처리가 성공한 경우의 처리
}
```  
  
반환 값이 3개라는 것은 동기적으로 함수 호출하는 경우는 거기까지 안 더러워지는데 goroutine와 channel을 거치면 커지게 됩니다.  
자신이 라이브러리나 외부에 제공한 함수를 장착할 때는 비동기의 경우에 어떻게 될지 의식해서 두면 좋다고 생각합니다.  
    
      

## 폴링

```
package main

import (
    "fmt"
    "time"
)

func main() {
    q := make(chan struct{}, 2)

    go func() {
        // 무거운 처리
        time.Sleep(3 * time.Second)
        q <- struct{}{} // 위의 make에서 버퍼 크기가 2 이므로
    }()

    for {
        if len(q) > 0 { // len은 channel에 담긴 queue 수를 반환. atomic
            break
        }

        // q 을 채울 때까지 다른 일을 한다
        time.Sleep(1 * time.Second)
        fmt.Println("何か")
    }
}
```
  
  
## Queuing
예를 들면 channel의 수신을 트리거로 하여 데이터베이스에 접속하여 갱신 처리를 실행하는 처리를 구현하려고 한다.  
복수의 트리거가 동시에 발생한 경우 channel 에는 갱신 리퀘스트가 쌓이지만 가능하면 1회 데이터베이스 접속으로 채널에 담긴 모든 것의 갱신 요청을 처리하려는 케이스도 있다. time.After와 조합한다.  

```
package main

import (
    "fmt"
    "time"
)

func main() {
    q := make(chan string, 5)

    go func() {
        time.Sleep(3 * time.Second)
        q <- "foo"
    }()
    go func() {
        time.Sleep(3 * time.Second)
        q <- "bar"
    }()

    // 거의 동시에 2개의 트리거가 발생한다
    var cmds []string
    cmds = append(cmds, <-q) // 먼저 하나씩 수신한다
wait_some:
    for {
        select {
        case cmd := <-q:
            // 1초 이내라면 함께 처리한다
            cmds = append(cmds, cmd)
        case <-time.After(1 * time.Second):
            // 1초 넘는다면 이제 받지 않도록 한다
            break wait_some
        }
    }
    for _, cmd := range cmds {
        fmt.Println(cmd)
    }
}
```
  
  
## Worker
일 양이 5개 있고 이것을 3개의 워커 스레드로 일을 처리하고 싶은 경우  
```
package main

import (
    "fmt"
    "sync"
    "time"
)

func fetchURL(wg *sync.WaitGroup, q chan string) {
    // 주의점: ↑ 이것이 포인터
    defer wg.Done()
    for {
        url, ok := <-q // close 되면 ok가 false 로 된다
        if !ok {
            return
        }

        fmt.Println("다운로드: ", url)
        time.Sleep(3 * time.Second)
    }
}
func main() {
    var wg sync.WaitGroup

    q := make(chan string, 5)

    // 워커를 3개 만든다
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go fetchURL(&wg, q)
    }

    // 동시에는 3개까지만 처리할 수 있다
    q <- "http://www.example.com"
    q <- "http://www.example.net"
    q <- "http://www.example.net/foo"
    q <- "http://www.example.net/bar"
    q <- "http://www.example.net/baz"
    close(q) // 이것은 중요하가

    wg.Wait() // 모든 goroutine 이 종료 될 때까지 기다린다
}
```  
  

## 세마포어 처럼 사용하기

```
package main

import (
    "fmt"
    "sync"
    "time"
)

var ch chan int = make(chan int, 4) // 병렬도를 4로 제한

func heavyFunc(i int) {
    ch <- 1 // 채널의 버퍼가 가득찼다면 블럭한다
    defer func() { <-ch }()

    fmt.Println("start:", i)
    time.Sleep(time.Second)
    fmt.Println("end:", i)
}

func main() {
    wg := sync.WaitGroup{}
    for i := 0; i < 20; i++ {
        wg.Add(1)
        go func(i int) {
            heavyFunc(i)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```
  
     
## 참고 글
- [chat 프로그램에서 go channel 을 안정적으로 사용하기](http://ahnseungkyu.com/237)
  