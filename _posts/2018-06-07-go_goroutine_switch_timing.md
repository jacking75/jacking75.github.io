---
layout: post
title: golang - goroutine이 교체 되는 타이밍
published: true
categories: [Golang]
tags: go 번역 goroutine
---
## 결론
Go언어에서 goroutine은 반드시 스위치 하는 것은 아니다.  
"for() {}" 같은 비지 루프를 GOMAXPROCS의 지정 수 이상 포함시키면 스위칭 되지 않는다.  
  
    
## 경위
처리가 없는 비지 루프가 있으면 goroutine이 교환 되지 않고 처리가 멈추는 것을 알았다.  
처리 내용을 전부 주석 처리 하고 디버깅 하다가 알았다.  
  
test1.go  
```
package main

func busy() {
    for {
    }
}

func main() {
    go busy()

    go func() {
        println("hello")
    }()

    i := make(chan int)
    <-i
}
``` 
  
[StackOverflow의 글](http://stackoverflow.com/questions/17953269/go-routine-blocking-the-others-one)에 따르면  
- Go의 goroutine 스케쥴러는 논 프리엔티브(협조형 멀티 태스킹)이다.
- 주로 아래의 계기로 스케줄에 제어가 넘어가 스위치가 일어난다.
    - unbuffer한 채널에 대한 읽기
    - 시스템 콜 호출
    - 메모리 할당
    - time.Sleep() 호출
    - runtime.Gosched() 호출  
  	
또 다시 busy()를 아래처럼 바꾸어서 테스트 해보았다.  
```
func busy() {
    for i:=0; i<1e8; i++ {
    }
}
```
output
<pre>
hello
fatal error:all goroutines are asleep-deadlock!
</pre>  
  
위의 경우 고루틴이 스위치된 이유는 Go 1.2에 추가된 기능으로 인라인 함수가 아니면 스위치 되도록 되었기 때문이다.
만약 busy()가 인라인 된다면 스위치 되지 않는다. 단 busy()가 인라인 된다고 해도 스위치는 된다.   
이유는 루프 계산을 하기 때문에 변수를 만지고 즉 이것은 스택을 만지기 때문에 스위치 된다.  
만약 busy()가 인라인 되고 이 함수 안의 루프가  for() {} 라면 스위치 되지 않는다.  
  
      
<br>
<br>  

출처: https://qiita.com/umisama/items/93333ffe4d9fc7e4ba1f