---
layout: post
title: golang - 배열과 slice 기초
published: true
categories: [Golang]
tags: go slice array
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
## 배열 
  
### 배열 기초
- 배열 사용
```
// 크기 4, int 형 배열
var a [4]int

// 0번째 요소에 1 대입
a[0] = 1

// 변수 i를 선언하고 a[0]을 i에 대입
i := a[0]
```  
  
- 배열은 명시적인 초기화는 필요하지 않다.
- 배열 변수는 배열 전체를 가리키고 있으며, C 언어처럼 선드 배열 요소를 가리키는 포인터가 아니다.
- 배열 만들기
    - 요소 수를 지정한다. b := [2]string{ "aaa", "bbb" }
	- 요소 수를 지정하지 않는다. b := [...]string{ "aaa", "bbb" }
- 함수에 인자로 넘기면 기본적으로 값 전달이 된다. 참조 전달로 넘기고 싶다면 포인터를 사용해야 한다.
```
func fnV(arr [4]int) {
	for i, _ := range arr {
		arr[i] = 0
	}
	log.Println(arr) // [0, 0, 0, 0]
}

func fnP(arr *[4]int) {
	for i, _ := range arr {
		arr[i] = 0
	}
	log.Println(arr) // [0, 0, 0, 0]
}

func main() {
	arr := [4]int{1, 2, 3, 4}
	fnV(arr)
	log.Println(arr) // [1, 2, 3, 4]
	fnP(&arr)
	log.Println(arr) // [0, 0, 0, 0]
}
```
    
  
