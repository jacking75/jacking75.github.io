---
layout: post
title: golang - 성능 조사 slice vs array
published: true
categories: [Golang]
tags: go slice array
---
[출처](http://qiita.com/oywc410/items/1c4e4c2c5744491251b6)  
테스트 환경: go version go1.7.3 windows/amd64  
  
```
package go1_6

const capacity = 1024

//배열 조작 array는 slice 보다 빠르다
func array() [capacity]int { // 배열 처리 테스트 함수
    var d [capacity]int

    for i := 0; i < len(d); i++ {
        d[i] = 1
    }

    return d
}

func slice() []int { // slice 처리 테스트 함수
    d := make([]int, capacity)

    for i := 0; i < len(d); i++ {
        d[i] = 1
    }

    return d
}

//파라메터로서 넘긴 경우는 slice쪽이 빠르다(array는 메모리 복사가 발생하므로)
func inArray(array [capacity]int) { // 배열로서 함수 파라메터 효율 테스트
}

func inArray2(array *[capacity]int) { // 배열 포인터로 함수 파라메터 효율 테스트
}

func inSlice(slice []int) {//slice로 함수 파라메터 효율 테스트
}

//reArray 보다 빠르다
func reSlice() []byte { // return slice 효율 테스트
    return []byte{1, 2, 3, 4}
}

func reArray() [4]byte { // return array 효율 테스트
    return [4]byte{1, 2, 3, 4}
}
```

```
package go1_6

import (
    "testing"
)

func BenchmarkArray(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = array()
    }
}

func BenchmarkSlice(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = slice()
    }

}

func BenchmarkInArray(b *testing.B) {
    arr := array()
    for i := 0; i < b.N; i++ {
        inArray(arr)
    }
}

func BenchmarkInArray2(b *testing.B) {
    b.StopTimer()
    arr := array()
    b.StartTimer()

    parr := &arr
    for i := 0; i < b.N; i++ {
        inArray2(parr)
    }
}

func BenchmarkInSlice(b *testing.B) {
    b.StopTimer()
    sli := slice()
    b.StartTimer()

    for i := 0; i < b.N; i++ {
        inSlice(sli)
    }

}

func BenchmarkReArray(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = reArray()
    }
}

func BenchmarkReSlice(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = reSlice()
    }
}
```


< 테스트 결과 >
<img src="https://qiita-image-store.s3.amazonaws.com/0/138880/f7aea32e-5ff0-63cb-ac1e-91197b7c2996.png">

### 포인트
- 일반적인 처리 때는 배열이 Slice 보다 더 빠르지만 배열은 함수 매개 변수와 함수 반환 값으로 이용된 경우 메모리에 복사되기 때문에 오히려 효율이 떨어진다.  
    
 
