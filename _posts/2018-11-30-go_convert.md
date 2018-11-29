---
layout: post
title: golang - 변환
published: true
categories: [Golang]
tags: go 변환 convert
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
[출처](http://matope.hatenablog.com/entry/2014/04/22/101127)  
문자열 <-> 수치 변환  
strconv 패키지를 사용한다.  
  
## 문자열 → 수치변환
- func Atoi(s string) (i int, err error)  
문자열을 10 진수 int 타입으로 변환한다. ParseInt(s, 10, 0)의 간략화 버전  
```
var i int
i, _ = strconv.Atoi("255")
fmt.Println(i)  // => 255
```
  
- func ParseBool(str string) (value bool, err error)  
문자열을 bool 타입으로 변환한다.  
```
var b bool
b, _ = strconv.ParseBool("true")
fmt.Println(b) // => true
```  
받아 들일 수 있는 값은 1, t, T, TRUE, true, True, 0, f, F, FALSE, false, False  
  
- func ParseInt(s string, base int, bitSize int) (i int64, err error)  
문자열을 임의의 기수(2진수 ~ 36진수), 임의의 비트 길이(8〜64bit)의 Int로 변환  
```
var i32, i64, ib16, ib0 int64
i32, _ = strconv.ParseInt("255", 10, 32)
i64, _ = strconv.ParseInt("255", 10, 64)
ib16, _ = strconv.ParseInt("ff", 16, 16)
ib0, _ = strconv.ParseInt("0xff", 0, 16)
fmt.Println(i32, i64, ib16, ib0)   // => 255 255 255 255
```  
base는 변환에 사용할 기수(2〜36). 0의 경우 s 문자열 서식으로 판단한다(0x 접두어가 붙어 있다면 16진수 등). bitSize는 0,8,16,32,64(각각 int, int8, int16, int32, and int64에). 어떤 bitSize 라도 반환 값의 형 자체는 int64이지만 각각의 타입으로 값을 변하지 않도록 변환 할 수 있다.  

- func ParseUint(s string, base int, bitSize int) (n uint64, err error)  
문자열을 임의의 기수(2진수〜36진수), 임의의 비트 길이(8〜64bit)의 Uint로 변환  
```
var ui uint64
ui, _ = strconv.ParseUint("255", 10, 32)
fmt.Println(ui) // => 255
```
  
- func ParseFloat(s string, bitSize int) (f float64, err error)  
문자열을 임의의 비트 길이(32,64bit)의 Uint로 변환  
bitSize는 32 혹은 64(float32와 float64). 반환 값의 타입 자체는 float64이지만 각각의 타입에 값이 변하지 않도록 변환할 수 있다.  
```
var f32, f64 float64
f32, _ = strconv.ParseFloat("3.14159265359", 32)
f64, _ = strconv.ParseFloat("3.14159265359", 64)
fmt.Println(f32, f64) // => 3.1415927410125732 3.14159265359
```
  
- ParseXXX 함수의 에러 핸들링  
ParseXXX 함수의 error 타입은 NumError로 (NumError).Err로 범위 에러인지 혹은 서식 에러인지 알 수 있다.  
```
_, e = strconv.ParseInt("Bad number", 10, 32)
if e != nil {
if enum, ok := e.(*strconv.NumError); ok {
      switch enum.Err {
      case strconv.ErrRange:
        log.Fatal("Bad Range Error")
      case strconv.ErrSyntax:
        log.Fatal("Syntax Error")
      }
    }
}
``` 

  
  
## 수치 → 문자열(포맷)

```
func Itoa(i int) string
func FormatBool(b bool) string
func FormatInt(i int64, base int) string
func FormatUint(i uint64, base int) string
```
  
각 수치 타입을 문자열로 포맷한다.  
```
fmt.Println(strconv.Itoa(255)) // => 255
fmt.Println(strconv.FormatBool(true)) // => true
fmt.Println(strconv.FormatInt(255, 10)) // => 255
```
  
- func FormatFloat(f float64, fmt byte, prec, bitSize int) string  
float64 타입을 문자열로 포맷한다.  
```
fmt.Println(strconv.FormatFloat(1234.56789, 'b', 4, 64)) // => 5429687001335527p-42
fmt.Println(strconv.FormatFloat(1234.56789, 'e', 4, 64)) // => 1.2346e+03
fmt.Println(strconv.FormatFloat(1234.56789, 'E', 4, 64)) // => 1.2346E+03
fmt.Println(strconv.FormatFloat(1234.56789, 'f', 4, 64)) // => 1234.5679
fmt.Println(strconv.FormatFloat(1234.56789, 'g', 4, 64)) // => 1235
fmt.Println(strconv.FormatFloat(1234.56789, 'G', 4, 64)) // => 1235
fmt.Println(strconv.FormatFloat(1234.56789, 'G', 3, 64)) // => 1.23E+03
```  

- fmt는 출력 형식으로 'b','e','E','f','g','G' 중에서  
    - b는 이진 대수 표기
    - e와 E는 10 진수 대수 표기
    - f는 비 대수 표기
    - g,G는 큰 수는 대수, 그렇지 않으면 비 대수적으로 표기
  
prec는 점수. eEF 때 소수점 이하 점수. gG 때 전체 점수.(g,G는 prec가 f의 소수점 이상의 점수를 만족하고 있다면 f 또는 F로 되는 듯)  
  

문자열화 할 때는 fmt.Sprint(foo) 방식으로 할 수도 있다.  
  

  
## string => slice
출처: https://github.com/soshi8/GoLangTrans  
```
package main

import (
	"fmt"
)

func main() {

	var aaa string
	aaa = "sample string"
	bbb := []byte(aaa)

	fmt.Printf("bbb = %s\n", bbb)
}
```

  
## slice => string  

```
package main

import (
	"fmt"
)

func main() {

	var aaa []byte
	//	aaa = []byte("sample string")
	aaa = []byte{115, 97, 109, 112, 108, 101}
	bbb := string(aaa)

	fmt.Printf("bbb = %s", bbb)
}
```

  
## slice => array

```
package main

import (
	"fmt"
)

func main() {

	var aaa [6]byte
	var bbb []byte
	aaa = [6]byte{115, 97, 109, 112, 108, 101}
	bbb = aaa[:]

	fmt.Printf("bbb = %s", bbb[:])
}
```  


## array => slice

```
package main

import (
	"fmt"
)

func main() {

	var aaa []byte
	var bbb = make([]byte, 6)
	aaa = []byte{115, 97, 109, 112, 108, 101}
	copy(bbb, aaa[:])

	fmt.Printf("bbb = %s", bbb)
}
```
  
  
  
## uint16 => []byte (Little Enddian)

```
package main

import (
	"encoding/binary"
	"fmt"
)

func main() {

	var aaa uint16 = 4011
	bbb := make([]byte, 4)

	binary.LittleEndian.PutUint16(bbb, aaa)
	fmt.Printf("LE1:%x\n", bbb)
}
```

  
## []byte => Uint32 (Little Enddian)

```
package main

import (
	"encoding/binary"
	"fmt"
)

func main() {

	aaa := []byte{115, 97, 109, 112}

	bbb := binary.LittleEndian.Uint32(aaa)
	fmt.Printf("LE1:%x\n", bbb)
}
```    