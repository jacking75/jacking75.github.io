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
    
## 슬라이스
  
### 슬라이스 만들기
- 가변 길이 배열.
- 고정 길이 배열의 view로 구현 되어 있다.
- 슬라이스 만들기
    - letters := []string{ "a", "b", "c" }
	- 요소 수 지정 이외에는 배열과 만드는 방법은 비슷
- make 함수를 사용하여 만들기
    - func make([T], len, cap) []T
	- T: 요소 타입
	- len: 길이
	- cap: 용량(옵션)
- make 함수는 슬라이스 타입 오브젝트를 확보하고 초기화 한다.
- 첫번째 인수의 타입과 같은 타입을 반환한다.
- 포인터를 반환하는 것이 아니다.
- make 함수를 사용하여 슬라이스 만들기  
  
```
var s []byte
s = make([]byte, 5, 5)
// default로 초기 값 설정된다.
// s = [ 0 0 0 0 0 ], s의 타입 = []unit8
```
  
  
### 용량
- 용량(capacity)은 생략 할 수 있다.  
  
```
var s []byte
s = make([]byte, 5)
```  
  
- 용량을 생략하면 길이 == 용량이 된다.
- 슬라이스의 길이와 용량을 조사할 때는 len과 cap를 사용한다.  
  
```
len(s) // == 5
cap(s) // == 5
```
  
  
### 미 초기화
- 초기화 하지 않은 슬라이스의 값은 nil
- 슬라이스가 == nil 이라면 len, cap 함수는 0을 반환한다.
  
    
