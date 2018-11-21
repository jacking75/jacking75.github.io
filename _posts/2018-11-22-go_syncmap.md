---
layout: post
title: golang - sync.Map
published: true
categories: [Golang]
tags: go sync.Map
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  

## sync.Map의 성능  
[원문](https://tanksuzuki.com/entries/golang-syncmap/)  
  
Golang 동시성은 공유 리소스에 액세스 하는 경우 충돌이 발생하지 않도록 잠금을 준비해야 한다.  
예를 들어 표준 맵의 경우 goroutine safe 하지 않기 때문에, Write 끼리 충돌하거나 Write 중에 Read가 발생한 경우 Panic을 일으킨다.  
따라서 기존 맵과 sync.RWMutex를 함께 사용하여 잠금을 준비하고 여러 goroutine에서 안전하게 액세스 할 수 있도록 하는 것이 정석이었다.  
  
Go1.9에서 sync.Map이 표준 패키지에 들어와서 이제 sync.RWMutex를 쓰지 않아도 잠금 장치가 있는 맵을 사용할 수 있게 되었다.  
  

## 사용법

```
package main

import (
	"fmt"
	"sync"
)

func main() {
	// Map를 만든다
	m := sync.Map{}

	// 보존
	m.Store("hoge", "fuga")

	// 취득
	// 표준 Map과 같이 key가 존재하지 않으면 ok == false가 된다
	if v, ok := m.Load("hoge"); ok {
		fmt.Printf("hoge: %s\n", v) // hoge: fuga
	}

	// 신규 key의 경우는 등록+취득
	// 기존 key의 경우는 취득만(loaded == true로 된다)
	actual, loaded := m.LoadOrStore("hoge", "piyo")
	fmt.Printf("hoge: %s, loaded=%t\n", actual, loaded) // hoge: fuga, loaded=true

	// 맵 중의 모든 요소를 참조한다
	// 인수로 넘긴 함수가 false를 반환하면 Range를 종료한다
	m.Range(func(k, v interface{}) bool {
		fmt.Printf("key: %s, value: %s\n", k, v) // key: hoge, value: fuga
		return true
	})

	// 삭제
	m.Delete("hoge")
}
```  
  

## 측정  
sync.Map은 종래의 sync.RWMutex한 맵과 달리 내부에 2 종류의 맵을 사용하고 있다.  
  
### readOnly
- 잠금이 불필요한 항목을 저장(캐시 역할을 담당)
- atomic으로 제어되며, readOnly에 있는 항목이면 잠금없이 참조 · 갱신이 가능
- readOnly에 존재하지 않는 key의 처리 요구를받은 경우 dirty를 참조

  
### dirty
- 잠금이 필요한 항목을 저장한다
- mutex로 제어되고 잠금이 필요한 작업은 dirty에서 실시
- readOnly 캐시 미스가 다발하면 dirty는 readOnly화 된다  
  
[공식 벤치 마크](https://github.com/golang/go/blob/release-branch.go1.9/src/sync/map_bench_test.go)에서 sync.Map뿐만 아니라 sync.RWMutex 맵도 포함되어 있어 양자를 비교할 수 있다. 또한 sync.Map이 반드시 우위라는 것은 없다.  
  
**sync.Map은 Load는 빠른하지만 Store는 sync.RWMutex보다 약간 느리다. 특히 키가 분산되면 dirty 측의 처리가 많아지면서 readOnly의 atomic 효과가 없어 지기 때문에 더블 레이어 구조에 의한 오버 헤드가 발생한다.**  
  
아래은 공식 벤치 결과와 메모이다. 환경은 Sierra, 1.1 GHz Intel Core m3 8 GB.  

  
### Load
적중률에 관계없이 sync.Map이 더 빠르고. readonly 잠금 없이 얻을 수 있기 때문이라고 생각한다.
- 1023/1024의 확률로 값이 존재하는 경우에 Load 하는 경우
  
<pre>
BenchmarkLoadMostlyHits/*sync_test.RWMutexMap-4          	20000000	       102 ns/op	       7 B/op	       0 allocs/op
BenchmarkLoadMostlyHits/*sync.Map-4                      	30000000	        47.5 ns/op	       7 B/op	       0 allocs/op
</pre>  
  
- 1/1024의 확률로 값이 존재하는 경우에 Load 하는 경우  
  
<pre>
BenchmarkLoadMostlyMisses/*sync_test.RWMutexMap-4        	20000000	        90.5 ns/op	       7 B/op	       0 allocs/op
BenchmarkLoadMostlyMisses/*sync.Map-4                    	30000000	        35.7 ns/op	       7 B/op	       0 allocs/op
</pre>  

  
### LoadOrStore
Store 될 가능성이 있으면 sync.RWMutex가 더 빠르다.  
기존 key의 LoadOrStore(제일 아래 결과)는 sync.Map쪽이 빠르지만, 이것은 반환 값을 readOnly에서 잠금없이 얻을 수 있기 때문이다.  
  
- 128/256의 확률로 값이 존재하는 경우에 LoadOrStore 하는 경우  
<pre>
BenchmarkLoadOrStoreBalanced/*sync_test.RWMutexMap-4     	 2000000	       833 ns/op	      97 B/op	       2 allocs/op
BenchmarkLoadOrStoreBalanced/*sync.Map-4                 	 2000000	      1110 ns/op	      89 B/op	       3 allocs/op
</pre>  
  
- 정의되지 않은 키를 LoadOrStore( m.LoadOrStore(i, i))  
<pre>
BenchmarkLoadOrStoreUnique/*sync_test.RWMutexMap-4       	 1000000	      1123 ns/op	     178 B/op	       2 allocs/op
BenchmarkLoadOrStoreUnique/*sync.Map-4                   	 1000000	      1444 ns/op	     163 B/op	       4 allocs/op
</pre>  
  
- 기존의 키를 LoadOrStore( m.LoadOrStore(0, 0))  
<pre>
BenchmarkLoadOrStoreCollision/*sync_test.RWMutexMap-4    	10000000	       150 ns/op	       0 B/op	       0 allocs/op
BenchmarkLoadOrStoreCollision/*sync.Map-4                	50000000	        25.2 ns/op	       0 B/op	       0 allocs/op
</pre>  

  
### Range
sync.Map이 더 빠르다. 지금까지의 결과와 마찬가지로 readOnly를 참조하기 때문.  
  
그러나 readOnly와 dirty에 차이가 있을 경우, dirty가 readOnly로 승격 한 후에 Range가 실행된다([sync/map.go#L315](https://github.com/golang/go/blob/release-branch.go1.9/src/sync/map.go#L315)).  
**빈번한 Store 의해 readOnly와 dirty에 차이가 나기 쉬운 경우에는 성능이 저하 될 수 있을 것이다.**  
  
- 요소 1024의 맵을 Range 에서 참조  
<pre>
BenchmarkRange/*sync_test.RWMutexMap-4                   	   20000	     94268 ns/op	   16384 B/op	       1 allocs/op
BenchmarkRange/*sync.Map-4                               	  100000	     15142 ns/op	       0 B/op	       0 allocs/op
</pre>  
  
    
## 정리
- sync.Map에 의해 sync.RWMutex을 쓰지 않아도 잠금 장치가 있는 맵을 사용할 수 있다
- key가 분산하는 Store가 많은 경우는 sync.Map 보다 기존의 sync.RWMutex 쪽이 성능이 더 나올 가능성이 있다
- Load 대해서는 잠금이 발생하지 않는 sync.Map쪽이 2 ~ 3배 빨랐다