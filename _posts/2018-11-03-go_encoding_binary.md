---
layout: post
title: golang - binary encoding
published: true
categories: [Golang]
tags: go binaryencoding encoding 
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
## 바이너리 엔코딩
  
### 개요
- 복수의 값과 바이트 열의 상호 교환을 한다
    - 복수의 값은 고정 길이 값으로 해석된다.
    - 고정 길이 값으로 다룰 수 있는 것
        - 수치 타입: int8, uint8, int16, float32, complex64, …
        - 배열
        - 고정 길이 값으로 구성된 구조체
    - 엔디언 지정 가능(빅엔디언, 리틀엔디언)
- varint 엔코드/디코드 하기
    - varint 라는 것은 그 값에 맞는 바이트 열 길이가 변하는 정수 값(사양: Protocol Buffer 에서 유래）
  
  
### Read와 Write
binary 패키지에서 주로 사용하는 함수는 Read와 Write이다.  
어떤 값을 byte 슬라이스에 변환하는 경우는 Read를 사용하고, byte 슬라이스에서 값을 돌아가는 경우는 Write를 사용한다. order를 지정하는 것으로 엔디언 지정이 가능하다.   

```
func Read(r io.Reader, order ByteOrder, data interface{}) error
func Write(w io.Writer, order ByteOrder, data interface{}) error
```  

- 예
    - Read
        - 빅엔디언: http://play.golang.org/p/ECSFzRrtx0
        - 리틀엔디언: http://play.golang.org/p/Utw-Q74GLU
    - Write
        - 빅엔디언: http://play.golang.org/p/RXVEwvrTcR
        - 리틀엔디언: http://play.golang.org/p/VkgfdNuLa9
data로 넘길 수 있는 타입은 Read의 경우 고정 길이 값의 포인터 또는 고정 길이 값으로 구성되는 슬라이스 이다.  
Write의 경우 이것에 더해서 고정 길이 값 그것을 넘길 수 있다.data에는 구조체도 넘길 수 있지만 고정 길이가 아니면 안된다.  
그래서 단체 슬라이스를 다룰 수 있는 것과 상관 없이 슬라이스를 포함한 구조체는 binary 패키지에서 다룰 수가 없다는 것을 주의해야 한다.  
  
  
### ByteOrder 인터페이스
바이트 오더 변환을 할 행동이 ByteOrder 인터페이스로서 정의 되어 있다.  
등호 없는 정수를 byte 슬라이스로 변환 하는 메소드(PutUintXX([]byte, uintXX)), byte 슬라이스를 등호 없는 정수로 변환하는 메소드 (UintXX([]byte) uintXX)를 가지고 있다.  

```
type ByteOrder interface {
    Uint16([]byte) uint16
    Uint32([]byte) uint32
    Uint64([]byte) uint64
    PutUint16([]byte, uint16)
    PutUint32([]byte, uint32)
    PutUint64([]byte, uint64)
    String() string
}
```  

리틀 엔디어과 빅 엔디어을 나타내는 ByteOrder 인터페이스 구현이 준비 되어 있다.  
http://golang.org/pkg/encoding/binary/#pkg-variables  
```
var BigEndian bigEndian
var LittleEndian littleEndian
```  
  
bigEndian 타입과 littleEndian 타입이 무엇인지 보면 빈 구조체로서 정의 되어 있다.  
http://golang.org/src/pkg/encoding/binary/binary.go  
```
type bigEndian struct{}
type littleEndian struct{}
```  
  
이것에 대해서 ByteOrder 인터페이스 메소드가 정의 되어 있다.  
Read() 및 Write()에 넘겨진 ByteOrder는 BigEndian 혹은 LittleEndian을 넘겨서 엔디언 지정을 한다.  
변태적인 엔디언 (미들엔디언 이라는 것이 있는 듯)을 사용하는 경우를 만나지 않는 이상 BigEndian 혹은 LittleEndian으로 충분하다.  
  
  
### 코드 보기
엔코더, 디코드 모두 reflect 패키지를 자주 사용하고 있다.  

#### 인코딩/디코딩

```
package helpers

import (
    "encoding/binary"
    "fmt"
    "strconv"
    "strings"
)

// Str2bytes converts string("00 00 00 00 00 00 00 00") to []byte
func Str2bytes(str string) []byte {
    bytes := make([]byte, 8)
    for i, e := range strings.Fields(str) {
        b, _ := strconv.ParseUint(e, 16, 64)
        bytes[i] = byte(b)
    }
    return bytes
}

// Bytes2str converts []byte to string("00 00 00 00 00 00 00 00")
func Bytes2str(bytes ...byte) string {
    strs := []string{}
    for _, b := range bytes {
        strs = append(strs, fmt.Sprintf("%02x", b))
    }
    return strings.Join(strs, " ")
}

// Bytes2uint converts []byte to uint64
func Bytes2uint(bytes ...byte) uint64 {
    padding := make([]byte, 8-len(bytes))
    i := binary.BigEndian.Uint64(append(padding, bytes...))
    return i
}

// Uint2bytes converts uint64 to []byte
func Uint2bytes(i uint64, size int) []byte {
    bytes := make([]byte, 8)
    binary.BigEndian.PutUint64(bytes, i)
    return bytes[8-size : 8]
}

// Bytes2int converts []byte to int64
func Bytes2int(bytes ...byte) int64 {
    if 0x7f < bytes[0] {
        mask := uint64(1<<uint(len(bytes)*8-1) - 1)

        bytes[0] &= 0x7f
        i := Bytes2uint(bytes...)
        i = (^i + 1) & mask
        return int64(-i)

    } else {
        i := Bytes2uint(bytes...)
        return int64(i)
    }
}

// Int2bytes converts int to []byte
func Int2bytes(i int, size int) []byte {
    var ui uint64
    if 0 < i {
        ui = uint64(i)
    } else {
        ui = (^uint64(-i) + 1)
    }
    return Uint2bytes(ui, size)
}
```  

       
#### 엔코더 사양
[encoding/binary/binary.go](encoding/binary/binary.go)  

```
func Write(w io.Writer, order ByteOrder, data interface{}) error {
    // Fast path for basic types.
    var b [8]byte
    var bs []byte
    switch v := data.(type) {
    case *int8:
        bs = b[:1]
        b[0] = byte(*v)
    case int8:
        bs = b[:1]
        b[0] = byte(v)
    case *uint8:
        bs = b[:1]
        b[0] = *v
    case uint8:
        bs = b[:1]
        b[0] = byte(v)
    case *int16:
        bs = b[:2]
        order.PutUint16(bs, uint16(*v))
    case int16:
        bs = b[:2]
        order.PutUint16(bs, uint16(v))
    case *uint16:
        bs = b[:2]
        order.PutUint16(bs, *v)
    case uint16:
        bs = b[:2]
        order.PutUint16(bs, v)
    case *int32:
        bs = b[:4]
        order.PutUint32(bs, uint32(*v))
    case int32:
        bs = b[:4]
        order.PutUint32(bs, uint32(v))
    case *uint32:
        bs = b[:4]
        order.PutUint32(bs, *v)
    case uint32:
        bs = b[:4]
        order.PutUint32(bs, v)
    case *int64:
        bs = b[:8]
        order.PutUint64(bs, uint64(*v))
    case int64:
        bs = b[:8]
        order.PutUint64(bs, uint64(v))
    case *uint64:
        bs = b[:8]
        order.PutUint64(bs, *v)
    case uint64:
        bs = b[:8]
        order.PutUint64(bs, v)
    }
    if bs != nil {
        _, err := w.Write(bs)
        return err
    }

    // Fallback to reflect-based encoding.
    v := reflect.Indirect(reflect.ValueOf(data))
    size, err := dataSize(v)
    if err != nil {
        return errors.New("binary.Write: " + err.Error())
    }
    buf := make([]byte, size)
    e := &encoder{order: order, buf: buf}
    e.value(v)
    _, err = w.Write(buf)
    return err
}
```  
  
정수형(등호 있는, 등호 없는 모두) 및 정수형의 포인터는  타입 switch 문에 의해서 판단되는 ByteOrder 인터페이스를 사용하여 byte 슬라이스로 변환 된다. 이 이외의 타입의 경우는 리플렉션을 사용하여 얻어진 타입 정보를 토대로 byte 슬라이스로 변환 된다.  
  
```
v := reflect.Indirect(reflect.ValueOf(data)) 
```  
를 하는 것으로  값이 포인터 타입 경우는 그 포인터가 가리키는 값의 reflect.Value 오브젝트를 얻는다. dataSize(v)로 값의 길이를 얻고, 그 길이만큼 byte 슬라이스를 생성하여 encoder에서 reflect.Value 오브젝트가 가리키는 값을 byte 슬라이스로 변환 한다.  

```
type coder struct {
    order ByteOrder
    buf   []byte
}

type encoder coder
```  

encoder는 ByteOrder 타입의 오브젝트와 byte 슬라이스에서 되는 구조체이다.  
```
func (e *encoder) value(v reflect.Value) {
    switch v.Kind() {
    case reflect.Array:
        l := v.Len()
        for i := 0; i < l; i++ {
            e.value(v.Index(i))
        }

    case reflect.Struct:
        t := v.Type()
        l := v.NumField()
        for i := 0; i < l; i++ {
            // see comment for corresponding code in decoder.value()
            if v := v.Field(i); v.CanSet() || t.Field(i).Name != "_" {
                e.value(v)
            } else {
                e.skip(v)
            }
        }

    case reflect.Slice:
        l := v.Len()
        for i := 0; i < l; i++ {
            e.value(v.Index(i))
        }

    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        switch v.Type().Kind() {
        case reflect.Int8:
            e.int8(int8(v.Int()))
        case reflect.Int16:
            e.int16(int16(v.Int()))
        case reflect.Int32:
            e.int32(int32(v.Int()))
        case reflect.Int64:
            e.int64(v.Int())
        }

    case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
        switch v.Type().Kind() {
        case reflect.Uint8:
            e.uint8(uint8(v.Uint()))
        case reflect.Uint16:
            e.uint16(uint16(v.Uint()))
        case reflect.Uint32:
            e.uint32(uint32(v.Uint()))
        case reflect.Uint64:
            e.uint64(v.Uint())
        }

    case reflect.Float32, reflect.Float64:
        switch v.Type().Kind() {
        case reflect.Float32:
            e.uint32(math.Float32bits(float32(v.Float())))
        case reflect.Float64:
            e.uint64(math.Float64bits(v.Float()))
        }

    case reflect.Complex64, reflect.Complex128:
        switch v.Type().Kind() {
        case reflect.Complex64:
            x := v.Complex()
            e.uint32(math.Float32bits(float32(real(x))))
            e.uint32(math.Float32bits(float32(imag(x))))
        case reflect.Complex128:
            x := v.Complex()
            e.uint64(math.Float64bits(real(x)))
            e.uint64(math.Float64bits(imag(x)))
        }
    }
}
```  
  
수치 타입 경우는 encoder의 대응하는 메소드(encoder.uint8() 등)을 사용하여 byte 슬라이스로 변환된다. 배열 및 슬라이스의 경우는 각 요소에 대해서 encoder.value()가 재귀적으로 호출된다.  
구조체의 경우는 필드에 따라서 동작이 바뀐다.  
- 설정 가능한 필드 혹은 공백 필드가 아닌 경우: 재귀적으로 encoder.value()가 호출된다
- 이 이외의 경우(즉 공백 필드의 경우）: 0으로 패딩 된다.
- 의문
    - 수치 타입의 경우 v.Kind()로 타입 switch한 후 v.Type().Kind() 하지만 얻어지는 값이 다른가?
  
  
#### 디코드 사양
encoding/binary/binary.go  

```
func Read(r io.Reader, order ByteOrder, data interface{}) error {
    // Fast path for basic types.
    if n := intDestSize(data); n != 0 {
        var b [8]byte
        bs := b[:n]
        if _, err := io.ReadFull(r, bs); err != nil {
            return err
        }
        switch v := data.(type) {
        case *int8:
            *v = int8(b[0])
        case *uint8:
            *v = b[0]
        case *int16:
            *v = int16(order.Uint16(bs))
        case *uint16:
            *v = order.Uint16(bs)
        case *int32:
            *v = int32(order.Uint32(bs))
        case *uint32:
            *v = order.Uint32(bs)
        case *int64:
            *v = int64(order.Uint64(bs))
        case *uint64:
            *v = order.Uint64(bs)
        }
        return nil
    }

    // Fallback to reflect-based decoding.
    var v reflect.Value
    switch d := reflect.ValueOf(data); d.Kind() {
    case reflect.Ptr:
        v = d.Elem()
    case reflect.Slice:
        v = d
    default:
        return errors.New("binary.Read: invalid type " + d.Type().String())
    }
    size, err := dataSize(v)
    if err != nil {
        return errors.New("binary.Read: " + err.Error())
    }
    d := &decoder{order: order, buf: make([]byte, size)}
    if _, err := io.ReadFull(r, d.buf); err != nil {
        return err
    }
    d.value(v)
    return nil
}
```  
  
정수형(부호 있는. 부호 없는 쌍방) 포인터는 타입 스위치문에 따른 판단된 ByteOrder 인터페이스를 이용하여 byte 슬라이스에서 대응하는 정수형 값으로 변환된다. 그 이외 타입의 경우 리플렉션을 사용하여 얻을 수 있는 타입 정보를 byte 슬라이스에서 주어진 형태로 변환된다. 이 때 포인터 타입이나 슬라이스가 아니면 오류를 반환한다. decoder.value()을 이용하여 디코딩이 된다.  
```
type decoder coder
```  

decoder는 encoder와 같게 coder 에서 정의 되어 있다.  
```
func (d *decoder) value(v reflect.Value) {
    switch v.Kind() {
    case reflect.Array:
        l := v.Len()
        for i := 0; i < l; i++ {
            d.value(v.Index(i))
        }

    case reflect.Struct:
        t := v.Type()
        l := v.NumField()
        for i := 0; i < l; i++ {
            // Note: Calling v.CanSet() below is an optimization.
            // It would be sufficient to check the field name,
            // but creating the StructField info for each field is
            // costly (run "go test -bench=ReadStruct" and compare
            // results when making changes to this code).
            if v := v.Field(i); v.CanSet() || t.Field(i).Name != "_" {
                d.value(v)
            } else {
                d.skip(v)
            }
        }

    case reflect.Slice:
        l := v.Len()
        for i := 0; i < l; i++ {
            d.value(v.Index(i))
        }

    case reflect.Int8:
        v.SetInt(int64(d.int8()))
    case reflect.Int16:
        v.SetInt(int64(d.int16()))
    case reflect.Int32:
        v.SetInt(int64(d.int32()))
    case reflect.Int64:
        v.SetInt(d.int64())

    case reflect.Uint8:
        v.SetUint(uint64(d.uint8()))
    case reflect.Uint16:
        v.SetUint(uint64(d.uint16()))
    case reflect.Uint32:
        v.SetUint(uint64(d.uint32()))
    case reflect.Uint64:
        v.SetUint(d.uint64())

    case reflect.Float32:
        v.SetFloat(float64(math.Float32frombits(d.uint32())))
    case reflect.Float64:
        v.SetFloat(math.Float64frombits(d.uint64()))

    case reflect.Complex64:
        v.SetComplex(complex(
            float64(math.Float32frombits(d.uint32())),
            float64(math.Float32frombits(d.uint32())),
        ))
    case reflect.Complex128:
        v.SetComplex(complex(
            math.Float64frombits(d.uint64()),
            math.Float64frombits(d.uint64()),
        ))
    }
}
```  
  
수치 타입의 경우는 decoder의 대응하는 메소드(decoder.uint8() 등)을 이용하여 얻어진 값을 리플렉션을 사용하고 있다. 배열 및 슬라이스의 경우에는 각 요소에 대해서 decoder.value()가 재귀적으로 호출된다.  
구조체의 경우는 필드에 의해서 동작이 바뀐다.  
- 설정 가능한 필드, 혹은 공백 필드가 아닌 경우: 재귀적으로 decoder.value()가 호출된다
    - 조건 분기는 최적화를 위해 StructField를 참조하지 않도록 하고 있다
- 그 이외의 경우(즉, 공백 필드의 경우): 길이만큼 건너 뛰고 읽는다
    - 변환되지 않는 필드가 있다면 panic이 발생:http://play.golang.org/p/ccsVEYuKA5
  
  
### Varint
[사양](https://developers.google.com/protocol-buffers/docs/encoding?hl=ja&csw=1)  
- 최후의 1 바이트를 제외하고 각 바이트의 최상 위 비트는 1
    - 각 바이트의 최상위 비트가 1이면, 후속의 바이트가 존재한다는 것을 밝힌다
- 각 바이트에 대해서 최상위 비트 제외한 7비트를 사용하고 2의 보수로 표현
- 수치의 하위 그룹(7비트 단위)이 하위 바이트가 되게 인코딩한다
  
  
#### 조작

```
func Varint(buf []byte) (int64, int)
func Uvarint(buf []byte) (uint64, int)

func PutVarint(buf []byte, x int64) int
func PutUvarint(buf []byte, x uint64) int

func ReadVarint(r io.ByteReader) (int64, error)
func ReadUvarint(r io.ByteReader) (uint64, error)
```  

Varint()과 Uvarint는 byte 슬라이스에서 정수 값을 얻는데 이용.  
PutVarint()과 PutUvarint()는 정수 값으로부터 byte 슬라이스를 얻는 데 이용한다.  
byte 슬라이스가 아닌 io.ByteReader를 사용하고 싶은 경우에는 ReadVarint()또는 ReadUvarint()을 이용한다.  
  
  

### 참고
- 문서: http://golang.org/pkg/encoding/binary/
- 소스 코드
    - [binary.go](https://code.google.com/p/go/source/browse/src/pkg/encoding/binary/binary.go)
    - [varint.go](https://code.google.com/p/go/source/browse/src/pkg/encoding/binary/varint.go)
  