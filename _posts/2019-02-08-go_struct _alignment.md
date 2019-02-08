---
layout: post
title: golang - 구조체의 정확한 크기 계산하기
published: true
categories: [Golang]
tags: go struct alignment
---
golang의 구조체도 c/c++과 같이 패딩이 있다.  
```
type PacketHeader struct {
	N1 int32 
	N2 int16
	N3 int64
}
```  

위 구조체를 unsafe.Sizeof를 하면 패딩 때문에 크기가 16이 나온다.  
(sizeof가 기본 기능이 아니고 unsafe 패키지를 사용해야 한다.)  

구글링을 해 봐도 #pragma pack(1) 같은 것으로 1바이트 정렬하는 기능이 없다. ;;;;  
그래서 구조체의 정확한 크기를 알기 위해 서는 reflect를 사용하여 아래와 같이 해야 한다.  

https://play.golang.org/p/3NbjRjSRk-y  

```
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func sizeof(t reflect.Type) int {
	switch t.Kind() {
	case reflect.Array:
		//fmt.Println("reflect.Array")
		if s := sizeof(t.Elem()); s >= 0 {
			return s * t.Len()
		}

	case reflect.Struct:
		//fmt.Println("reflect.Struct")
		sum := 0
		for i, n := 0, t.NumField(); i < n; i++ {
			s := sizeof(t.Field(i).Type)
			if s < 0 {
				return -1
			}
			sum += s
		}
		return sum

	case reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64,
		reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64,
		reflect.Float32, reflect.Float64, reflect.Complex64, reflect.Complex128:
		//fmt.Println("reflect.int")
		return int(t.Size())
	case reflect.Slice:
		//fmt.Println("reflect.Slice:", sizeof(t.Elem()))
		return 0
	}

	return -1

}


type PacketHeader struct {
	N1 int32 // sizeof에서 사용할 때 int로 하면 안 됨
	N2 int16
	N3 int64
}

func main() {
	var packetHeader PacketHeader
	
	packetHeaderSize1 := unsafe.Sizeof(packetHeader)	
	fmt.Println("PacketHeader size: ", packetHeaderSize1)
	
	// sizeof 함수를 사용하면 패딩 없는 정확한 크기를 알 수 있다
	packetHeaderSize2 := sizeof(reflect.TypeOf(packetHeader))	
	fmt.Println("PacketHeader size: ", packetHeaderSize2)
}

```