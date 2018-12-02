---
layout: post
title: golang - new와 make의 차이
published: true
categories: [Golang]
tags: go new make
---
[원문](http://cipepser.hatenablog.com/entry/go-new-make)  
  
## 정리
차이를 간략하게 설명한다.  
  
|    | new(T)                     | make(T)                 |
|--------|----------------------------|-------------------------|
| 대상   | 임의의 타입                   | slice, map, channel 만 |
| 초기화 | 초기화 하지 않는다(0 값으로 된다) | 초기화 한다              |
| 반환 값 | *T                         | T                       |

        
## 대상 및 초기화에 대해
new()와 make()에서 초기화 하지 않는/하는의 차이는 slice, map, channel이 내부 데이터 구조를 가지는 것에서 온다.  
  
아래에 runtime 패키지에서 각각 타입이 정의 되어 있는 부분을 인용.  
가장 이해하기 쉬운 것이 slice 이다.  
array (실제 데이터), len, cap을 초기화 해 줄 필요가 있기 때문에 make()가 준비 되어 있다.  


  
### Slice

```
type slice struct {
      array unsafe.Pointer
      len   int
      cap   int
  }
```  
https://golang.org/src/runtime/slice.go#L11-15  


  
### map

```
type hmap struct {
      // Note: the format of the Hmap is encoded in ../../cmd/internal/gc/reflect.go and
      // ../reflect/type.go. Don't change this structure without also changing that code!
      count     int // # live cells == size of map.  Must be first (used by len() builtin)
      flags     uint8
      B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
      noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
      hash0     uint32 // hash seed
  
      buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
      oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
      nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
  
      extra *mapextra // optional fields
  }
```  
https://golang.org/src/runtime/hashmap.go#L106-120  


  
### channel

```
type hchan struct {
      qcount   uint           // total data in the queue
      dataqsiz uint           // size of the circular queue
      buf      unsafe.Pointer // points to an array of dataqsiz elements
      elemsize uint16
      closed   uint32
      elemtype *_type // element type
      sendx    uint   // send index
      recvx    uint   // receive index
      recvq    waitq  // list of recv waiters
      sendq    waitq  // list of send waiters
  
      // lock protects all fields in hchan, as well as several
      // fields in sudogs blocked on this channel.
      //
      // Do not change another G's status while holding this lock
      // (in particular, do not ready a G), as this can deadlock
      // with stack shrinking.
      lock mutex
  }
```  
https://golang.org/src/runtime/chan.go#L31-50  
  

  
## 초기화에 대해 보충
Effective Go 에서는 new()는 초기화를 하지 않는, 즉 제로 값이 되는 것에 대해 helpful 하다고 언급 했다.  
이것은 제로 값 자체가 의미를 가지는 경우에는 초기화 하는 것이 의미가 있다는 것이다.  
  
Effective Go 에서 언급된 예이지만 sync.Mutex 에서는 다음과 같이 제로 값 자체가 unlock인 state를 표현한다.  
```
// A Mutex is a mutual exclusion lock.
// The zero value for a Mutex is an unlocked mutex.
//
// A Mutex must not be copied after first use.
type Mutex struct {
  state int32
    sema  uint32
}
```   
  
자신의 형태를 정의 할 때, NewMyType() 같은 생성자를 제공하는 경우가 많을 것이다.  
이 때 제로 값 자체에 의미를 갖게하고, new()와 결합하면 좋은 디자인이 될 것으로 생각한다.  
  