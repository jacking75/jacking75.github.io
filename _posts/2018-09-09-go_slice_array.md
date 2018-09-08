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
  
  
### re-slice
- 다시 슬라이스 할 수 있다.
```
b := []byte{ 'g', 'o', 'o', 'g', 'l', 'e'}
c := b[1:2] // c = []byte{ '0', 'o' } c는 슬라이스 b와 같은 메모리 영역
```
- b[:] => b
- b[:2] => []byte{'g', 'o'}
- b[2:] => []byte{'o', 'g', 'l', 'e'}
- 배열에서 슬라이스를 만든다.
```
x := [3] string { "aaa", "bbb", "ccc" }
s := x[:] // 슬라이스 s는 x의 기억영역을 참조
```
- 슬라이스는 배열 내의 연속된 영역의 차조이면서 아래 3개로 구성 되어 있다.
    - 배열의 포인터
    - 세그멘트의 길이(length)
    - 용량(capacity)
![](/images/2018/golang/0006.PNG) 
  
  
### 슬라이스의 내부 구조
- s := make([]byte, 5)
![](/images/2018/golang/0007.PNG)   
- s := s[2:4]
![](/images/2018/golang/0008.PNG)   
- 재 슬라이스된 요소를 변경하면 원 슬라이스의 요소도 변경된다.
![](/images/2018/golang/0009.PNG)  
- len < cap 의 경우 len을 cap까지 확장할 수 있다.
    - s = s[:cap(2)]
![](/images/2018/golang/0010.PNG)  
  
  
### copy
- 용량을 늘리면 확장할 수 있다.
- 용량(capacity)을 늘리려면
    - 새로 큰 용량의 슬라이스를 만든다.
    - 이 새로운 슬라이스에 원래의 슬라이스 내용을 복사한다.
    - ![](/images/2018/golang/0011.PNG)
- copy 함수를 사용하면 아래와 같이 할 수 있다.
    - ![](/images/2018/golang/0012.PNG)
    - func copy(dst, src []T) int
    - 원 슬라이스에서 복사할 슬라이스에 데이터를 복사한다.
    - 복사한 요소 수를 반환한다.
- copy 함수는 서로 다른 length 슬라이스도 복사할 수 있다.
    - ![](/images/2018/golang/0013.PNG)
  
  
### append
- 슬라이스에 데이터를 추가할 때는 append 함수를 사용한다.
- 사용 방법은 아래와 같다
```
func append(s []T, x ...T) [] T
```
- 처리 내용은 슬라이스 s의 마지막에 요소 x를 추가한다.
- 보다 큰 용량이 필요하다면 (자동적으로)확장한다.  
![](/images/2018/golang/0014.PNG) 
- 슬라이스에 슬라이스를 추가할 때는 ... 을 사용한다.  
![](/images/2018/golang/0015.PNG)  
- append 주의
golang의 append는 용량 이내라면 증폭하지 않고 반환 값을 돌려준다. 아래의 코드는 큰 실수가 될 수 있다.   
```
hoge := []int{}
for _, comment := range comments {
     hoge = append(hoge, comment.ID)
 }
```  
위의 코드는 큰 문제가 없이 보이지만 루프 마다 새로운 slice를 만들고 기존 값을 복사 후 기존에 할당한 것은 파괴한다. O(N^2)가 된다.  
아래 코드처럼 수정해야 한다.  
```
hoge := make([]int, len(comments))
 l := len(comments)
for i, j := l - 1, 0; i >= 0; i-- {
     hoge[i] = comments[j].ID
     j++
 } 
```  
  
  
### 삭제
- 슬라이스에서 어떤 요소를 삭제할 때는  
```
// 예를 들면 3을 삭제하고 싶은 경우
s := []int{1,2,3,4,5}
s = append(s[:2], s{3:]...)
//s == []int{1,2,4,5}
```  
- 슬라이스에 대한 delete 함수 같은 것은 없다. 
- 위 방법으로 삭제를 해도 용량은 줄어들지 않는다. 길이만 줄어든다.
- 용량까지 줄이고 싶다면 아래와 같이한다.
![](/images/2018/golang/0016.PNG)  
  
  
### 2차원 배열을 한번에 초기화하기
정확하게는 슬라이스의 슬라이스  
```
matrix := [][]float64{{0, 0}, {0, 1}, {1, 0}, {1, 1}}
```  
아래와 같음  
```
matrix := make([][]float64, 4)
matrix[0] = []float64{0, 0}
matrix[1] = []float64{0, 1}
matrix[2] = []float64{1, 0}
matrix[3] = []float64{1, 1}
```  
```
fmt.Println(matrix) // [[0 0] [0 1] [1 0] [1 1]]
```  
  
  
### 배열과 배열 연결
[[1,3,4][2,5,6]] 이 아닌 [1,3,4,2,5,6] 와 같이 연결하고 싶다.  
```
slice1 := []int{0, 1, 2, 3, 4}
slice2 := []int{55, 66, 77}
fmt.Println(slice1)
slice1 = append(slice1, slice2...) // The '...' is essential!
fmt.Println(slice1)
```  
  
  
### GC
슬라이스에서 보이는 값이 사라졌지만 내부 배열에 남아 있어서 GC 되지 않고 값이 남는 경우가 있다.  
예를 들면 슬라이스에서 지정된 인덱스 값을 삭제하고 그 만큼 앞으로 땡겨서 값을 채워 넣는 del() 함수를 생각해 보자.  
이미 내부 배열 내의 이동만으로 끝나면 다시 메모리를 확보할 필요는 없으므로 다음과 같이 한다.
```
func del(a[]int, i int)[]int<
	copy(a[i:], a[i+1:])
	a=a[:len(a)-1]
	return a
}

func main(){
	a:=[]int{1, 2, 3, 4, 5}
	a=del(a, 2)
	log.Println(a, len(a), cap(a)//[1 2 4 5] 4 5
}
```  
메모리의 Allocate도 없고 슬라이스 측에서 보면 값은 사라졌다.  
그러나 내부 배열은 어떨까? del()의 뒤는 다음과 같이 되어 있다.(실제로 a[:cap(a)]에서 확인 가능하다.
```
//[]int*
//|
//[5]int[1, 2, 4, 5, 5]
log.Println(a[:cap(a)])
```  
이렇게 슬라이스에서는 보이지 않은 값이 내부 배열에 남아 있는 경우가 있다. 예를 들면 큰 사이즈의 구조체의 참조를 가진 슬라이스 등의 경우 내부 배열에 참조가 남아 있는 것으로 GC 되지 않을 가능성도 있다.
만전을 기하기 위해서는 del()을 아래와 같이 끝의 값에 타입에 따른 제로 값(선언만 하고 초기화하지 않는 것으로 생성 가능)을 대입 하는 방법이 있다. int의 경우는 0 이지만 참조형의 슬라이스의 경우는 nil이 된다.
```
var zero int //제로 값

func del(a[]int, i int)[]int<
	copy(a[i:], a[i+1:])
	a[len(a)-1]=zero  //제로 값, nil 대입
	a=a[:len(a)-1]
	return a
}
```  
삭제하는 범위가 큰 경우에는 일단 다른 조각으로 대비하여 GC를 재촉할 수 있다. 이 부분은 아래 링크의 글을 참고해라.  
Slice Tricks: https://code.google.com/p/go-wiki/wiki/SliceTricks  
Arrays, slices(and strings):The mechanics of'append'-The Go Blog: http://blog.golang.org/slices  
Go Slices:usage and internals-The Go Blog: http://blog.golang.org/go-slices-usage-and-internals    
   
  
### sort.Slice
1.8 버전에 추가된 기능이다.  
다양한 타입을 받아들이는데 속도가 sort.Sort와 비슷하다.  
```
// 이전 sort.Sort 에서는 정렬 교환에 관한 조작을 정의하고 sort.Interface 로 넘겼다.
type ByValue []string
func (s ByValue) Len() int           { return len(s) }
func (s ByValue) Swap(i, j int)      { s[i], s[j] = s[j], s[i] }
func (s ByValue) Less(i, j int) bool { return s[i] < s[j] }
var sslice []string = ...
sort.Sort(ByValue(sslice))

// sort.Slice
var sslice []string = ...
sort.Slice(sslice, func(i, j int) bool { return sslice[i] < sslice[j] })
// 이후 Len 이나 Swap 에 상응하는 처리는 정의하지 않아도 괜찮을까?
```  
  
```
// []int
hoge := []int{6, 2, 7, 1, 8, 4, 5}
sort.Slice(hoge, func(i, j int) { return hoge[i] < hoge[j] })

// []string
piyo := []string{"6", "2", "7", "1", "8", "4", "5"}
sort.Slice(piyo, func(i, j int) { return piyo[i] < piyo[j] })
```  
  
sort.Slice의 첫 번째 인수는 interface{} 로 되어 있고, 내부에서 reflect 패키지를 사용하여 처리하고 있다.  
그래서 어떤 slice 라도 처리할 수 있다. 그런데 속도는 그리 느리지 않다.  
자세한 이유는 [(일어)sort.Slice 로 배우는 고속화 힌트](https://qiita.com/chimatter/items/f908507287fe2c7030e9)를 보자.  
