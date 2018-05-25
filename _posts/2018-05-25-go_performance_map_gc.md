---
layout: post
title: golang - 성능 조사 map과 GC
published: true
categories: [Golang]
tags: go 번역 map gc
---
테스트 환경: go version go1.7.3 windows/amd64  
  
## 기본 용량
golang의 map은 용량을 확장 할 때 원 데이터의 복사와 재 해시를 하기 때문에 map을 만들 때 용량을 설정하는 것이 좋다.  
```
package main

import "testing"

func test(m map[int]int) {
    for i := 0; i < 10000; i++ {
        m[i] = i
    }
}

func BenchmarkMap(b *testing.B) {
    for i := 0; i < b.N; i++ {
        b.StopTimer()
        m := make(map[int]int)
        b.StartTimer()

        test(m)
    }
}

func BenchmarkCapMap(b *testing.B) {
    for i := 0; i < b.N; i++ {
        b.StopTimer()
        m := make(map[int]int, 10000)
        b.StartTimer()

        test(m)
    }
}
```  
  
<img src="https://qiita-image-store.s3.amazonaws.com/0/138880/2b180af0-6f44-c83f-9d3b-5cae547ec602.png">    
  
  
## value vs pointer
```
package main

import (
    "runtime"
    "time"
)

const capacity = 500000

var d interface{}

func value() interface{} {
    m := make(map[int]int, capacity)

    for i := 0; i < capacity; i++ {
        m[i] = i
    }

    return m
}

func pointer() interface{} {
    m := make(map[int]*int, capacity)

    for i := 0; i < capacity; i++ {
        v := i
        m[i] = &v
    }

    return m
}

func main() {
    //GC가 동작하는 시간은 map key/value 쪽이 map key/pointer 보다 빠르다
    //그래도 큰 value가 많은(struct 등) 경우는 key/pointer를 추천한다.
    d = value() // d = pointer()

    for i := 0; i < 20; i++ {
        runtime.GC()
        time.Sleep(time.Second)
    }
}
```

value 함수를 이용하는 경우:
<img src="https://qiita-image-store.s3.amazonaws.com/0/138880/243e11ee-f430-3b80-1e20-aa9baef16fa2.png">
  
pointer 함수를 이용하는 경우:
<img src="https://qiita-image-store.s3.amazonaws.com/0/138880/55dc6775-73f0-ebe2-9e85-d9834103adac.png">
  
  
## map과 메모리 공간
```
package main

import (
    "runtime/debug"
    "time"
)

const capacity = 1000000

var dict = make(map[int][100]byte, capacity)

func test() {
    // 데이터 추가
    for i := 0; i < capacity; i++ {
        dict[i] = [100]byte{}
    }

    //데이터 해제 (메모리 해제는 아니다)
    for k := range dict {
        delete(dict, k)
    }

    dict = nil //메모리 해제
}

func main() {
    test()

    for i := 0; i < 20; i++ {
        debug.FreeOSMemory()
        time.Sleep(time.Second)
    }
}
```
  
dict = nil 을 하지 않은 경우
<img src="https://qiita-image-store.s3.amazonaws.com/0/138880/c2497276-22e5-d3b0-2d98-1d9135d97d96.png">
  
dict = nil 하는 경우
<img src="https://qiita-image-store.s3.amazonaws.com/0/138880/71359d8d-85e1-f50b-bdad-337946b7a45e.png">
  
      
    
<br>
<br>  

출처: http://qiita.com/oywc410/items/ad8baee00f039705a5c0