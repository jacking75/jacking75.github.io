---
layout: post
title: golang - 성능 튜닝하기
published: true
categories: [Golang]
tags: go 성능 튜닝
---
## 대전제
- 프로파일링을 취한 뒤 어떻게 최적화 할 것인가에 대한 이야기  
   **추측하지말고 계측하라**  
- 알고리즘이나 데이터 구조는 최적인 것을 선택하고 있다고 가정.  
손 재주로 최적화하는 것보다 알고리즘 자체를 바꾸는 쪽이 압도적으로 좋아진다.  
- 이 기사의 각 벤치 마크는 Go 1.4(go version go1.4 linux/amd64)에서 다음 명령에서 실행  

```
go test -run NONE -bench . -benchmem  
```

- 벤치 마크의 결과는 환경에 좌우되므로 어디까지나 필자 환경에서의 결과  
  

  
## 메모리 할당 횟수를 줄인다
Go의 코드 속에서 퍼포먼스 저하 비율이 가장 큰 것이 메모리의 할당이다.  
즉, 메모리의 할당 횟수를 줄이면 퍼포먼스가 대폭 개선.  

```
package bench_test

import "testing"

func BenchmarkMemAllocOndemand(b *testing.B) {
    n := 10
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        s := make([]string, 0)
        for j := 0; j < n; j++ {
            s = append(s, "alice")
        }
    }
}

func BenchmarkMemAllocAllBeforeUsing(b *testing.B) {
    n := 10
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        s := make([]string, 0, n)
        for j := 0; j < n; j++ {
            s = append(s, "alice")
        }
    }
}
```  
  
<pre>
BenchmarkMemAllocOndemand        1000000              1953 ns/op             496 B/op         5 allocs/op
BenchmarkMemAllocAllBeforeUsing  3000000               571 ns/op             160 B/op         1 allocs/op
</pre>  
  

  
## 메모리 사용량
- 값은 8 바이트 정렬로 두고 기본은 8바이트 길이 만큼 메모리를 전유.
- 포인터 변수는 64bit CPU에서 8바이트 길이.
- 인터페이스 타입 변수는 16바이트 길이(값+타입 식별)  
  
메모리 확보를 포함한 타입 컨버트는 타입 캐스트, 어세션 비하면 10배 이상 느리다.
같은 값인데 "메모리 확보를 포함한 타입 컨버트"를 여러 번 실시하는 경우 메모리 소비량은 늘어나지만 범용 변수 "interface{}"에 값을 간직하도록 참조하는 것이 속도를 유지할 수 있다.  
  
  

## 제로 메모리 할당
고빈도 조작의 메모리 할당 1과 제로의 간에는 커다란 속도 차이가 있다.  
가능하면 제로 메모리 할당화을 목표로 한다.(사전 확보나 pool 등 도입)  
메모리 할당을 회피할 수 없는 상황에서 pool 등은 너무 사용하지 않는 것이 좋다(pool 하기 위해서 메모리 할당이 늘어난다면 NG).  
string을 컨버트 없이 []byte에 추가하는 방법이 있지만 다른 방법에 비교해서 바보 같으므로 이 방법을 하지 안 할지 항상 검토하는 게 좋다.  
  
  
## map과 slice의 인덱스 접속
map의 색인 액세스는 slice에 비해 수십배 느리다.  
100건 이하의 경우 바이너리 서치에서 slice에서 목적의 값을 찾는 것이 빠르다.  
100 요소 넘는 정도에서 map의 접근 속도 일정의 혜택이 발휘된다.  
  
  
## Go 생성자
new(Type)이 가장 기본으로 빠르다.  
그 외 생성자 방법은 "new+ 알파"로 몇 나노 느리다.(고빈도로 메모리 확보하는 구조체는 new만으로 끝나는 디자인이 더 좋다)  
제로 값으로 의도하지 않는 동작을 하지 않는 설계를 권장.  
  
  
## Go의 메모리 할당
- 작은(수십 킬로바이트 이하) 메모리 확보와 파괴가 같은 스택에서 행해지는 경우는 heap을 사용하지 않는다.  
- 스택에서 처리되므로 메모리 할당 조작 수는 제로다.  
- 스택에 의한 메모리 조작은 heap보다 50배 정도 빠르다.  
  
  
### 슬라이스 메모리 사용량
슬라이스 값은 다음의 3 필드의 구조체에 상당한다(3x8=24 바이트 길이).
이와 별도로 실제 배열 데이터 메모리 부분의 메모리를 소비한다.
- 버퍼에 대한 포인터
- 데이터 길이
- capacity 길이  
  
슬라이스 값을 다른 변수에 바인드 하면 위의 24바이트의 복사를 한다.  
미미한 개선에 불과하지만 슬라이스를 넘길 때 슬라이스로의 포인터 형을 사용하면 포인터 8 바이트의 복사만으로 전달한다.  
  
  
## 문자열의 결합 & Writer로의 출력
"bytes=append(bytes, str)"가 최고 속도.  
벤치: https://play.golang.org/p/10NVBfz2DW  
  
  
### fmt.Fprint와 []byte 변환
위 벤치, "w.Write([]byte(str)" 이 아닌 "fmt.Fprint(w, str)"는 후자 쪽이 빠르다.  
[]byte로의 변환은 메모리 확보와 복사가 발생하여 의외로 무거운 것 같다.  
  
"buff=buff[0:0]"는 슬라이스의 능력을 유지한 채 데이터 길이를 없애는 방법이다.  
  
  
## 덮어 쓰기와 defer
defer 경유와 defer 없이에서 5배 정도 속도 차이가 있다.  
그렇지만 defer 없이를 사용하는 것은 에워싸는 코드에서 return과 break, panic 등으로 빠져나가서 함수가 종료할 수 없는 것을 확신하는 경우와 속도가 엄격한 경우에만 둔다.  
속도가 중요하지 않거나 자신의 관리 밖의 코드가 사이에 있는 경우는 defer를 사용해야한다.  
 [벤치](https://play.golang.org/p/b5ZM1BW14M)  
  

  
## 요소 수를 사전에 알고 있는 경우에는 append를 사용하지 않는다
슬라이스에 요소를 채워 놓은 경우 append 에서 추가하기 보다 인덱스를 사용하여 슬라이스에 대입하는 것이 빠르다.  

```
package bench_test

import "testing"

func BenchmarkFillSliceByAppend(b *testing.B) {
    n := 100
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        s := make([]int, 0, n)
        for j := 0; j < n; j++ {
            s = append(s, j)
        }
    }
}

func BenchmarkFillSliceByIndex(b *testing.B) {
    n := 100
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        s := make([]int, n)
        for j := 0; j < n; j++ {
            s[j] = j
        }
    }
}
```   

<pre>
BenchmarkFillSliceByAppend        500000              2694 ns/op             896 B/op         1 allocs/op
BenchmarkFillSliceByIndex         500000              2487 ns/op             896 B/op         1 allocs/op
</pre>  
  

  
## 가능하면 channel을 사용하지 않는다
channel을 사용한 배타 제어보다 sync.Mutex, sync.RWMutex를 사용하는 것이 빠르다.

```
package bench_test

import (
    "sync"
    "testing"
)

func BenchmarkExclusiveWithChannel(b *testing.B) {
    c := make(chan struct{}, 1)
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        c <- struct{}{}
        // do something.
        <-c
    }
}

func BenchmarkExclusiveWithMutex(b *testing.B) {
    mu := new(sync.Mutex)
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        mu.Lock()
        // do something.
        mu.Unlock()
    }
}
```  

<pre>
BenchmarkExclusiveWithChannel   20000000                70.2 ns/op             0 B/op         0 allocs/op
BenchmarkExclusiveWithMutex     100000000               21.1 ns/op             0 B/op         0 allocs/op
</pre>  
  

마찬가지로 동기 처리도 sync.WaitGroup을 사용하는 것이 빠르다.  
```
package bench_test

import (
    "sync"
    "testing"
)

func BenchmarkSyncWithChannel(b *testing.B) {
    n := 10
    c := make(chan struct{}, n)
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        for j := 0; j < n; j++ {
            go func() {
                // do something.
                c <- struct{}{}
            }()
        }
        for j := 0; j < n; j++ {
            <-c
        }
    }
}

func BenchmarkSyncWithWaitGroup(b *testing.B) {
    n := 10
    var wg sync.WaitGroup
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        wg.Add(n)
        for j := 0; j < n; j++ {
            go func() {
                // do something.
                wg.Done()
            }()
        }
        wg.Wait()
    }
}
```  

<pre>
BenchmarkSyncWithChannel          500000              3511 ns/op             160 B/op        10 allocs/op
BenchmarkSyncWithWaitGroup        500000              3086 ns/op             164 B/op        11 allocs/op
</pre>  
  

  
## 가능하면 함수를 호출하지 않는다
Go는 함수와 메소드 호출이 느리다. 몇줄 정도라면 컴파일러가 인라인 전개해 주지만 그렇지 않은 경우는 느리다.  
Denco의 코드를 예로 들면 재귀 호출하는 것이 깔끔하지만 goto를 쓰도록 해서 퍼포먼스를 올리고 있다( https://github.com/naoina/denco/blob/master/router.go#L167-L205).  
  
  
### 정규 표현은 스위스 아머 칼이 아니다
Go의 regexp 패키지는 Ruby와 Python 등과 달리 꼭 선형 시간으로 처리가 완료되는 알고리즘을 채용하고 있다.  
https://groups.google.com/forum/#!msg/golang-nuts/6d1maef4jQ8/yg4NY5qRtsMJ  
  
그러나 그 대신 무척 느려져서 다른 언어처럼 쉽게 생각하고 사용 하면 꽤 성능이 떨어진다. 그러니, 읽기 쉽다는 장점이 있지만 정말 속도가 필요한 때는 문자열의 매칭과 치환 등은 가급적 자체적으로 쓰도록 하자.  

```
package bench_test

import (
    "regexp"
    "testing"
)

func BenchmarkStringMatchWithRegexp(b *testing.B) {
    s := "0xDeadBeef"
    re := regexp.MustCompile(`^0[xX][0-9A-Fa-f]+$`)
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        re.MatchString(s)
    }
}

func BenchmarkStringMatchWithoutRegexp(b *testing.B) {
    s := "0xDeadBeef"
    isHexString := func(s string) bool {
        if len(s) < 3 || s[0] != '0' || s[1] != 'x' && s[1] != 'X' {
            return false
        }
        for _, c := range s[2:] {
            if c < '0' || '9' < c && c < 'A' || 'F' < c && c < 'a' || 'f' < c {
                return false
            }
        }
        return true
    }
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        isHexString(s)
    }
}
```  

<pre>
BenchmarkStringMatchWithRegexp      2000000               643 ns/op             0 B/op         0 allocs/op
BenchmarkStringMatchWithoutRegexp   30000000               55.0 ns/op           0 B/op         0 allocs/op
</pre>  
  

  
## 짧은 처리는 goroutine을 사용하지 않는 것이 좋다
goroutine을 실행하는 데도 비용이 들기 때문에 뭐든지 go를 붙이면 빨라지는 것은 아니므로 가벼운 처리일수록 시퀀셜 하게 처리하는 것이 빠르다.  

```
package bench_test

import (
    "sync"
    "testing"
)

func BenchmarkGoroutine(b *testing.B) {
    n := 10
    var wg sync.WaitGroup
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        wg.Add(n)
        for j := 0; j < n; j++ {
            go func() {
                wg.Done()
            }()
        }
        wg.Wait()
    }
}

func BenchmarkSequential(b *testing.B) {
    n := 10
    var wg sync.WaitGroup
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        wg.Add(n)
        for j := 0; j < n; j++ {
            func() {
                wg.Done()
            }()
        }
        wg.Wait()
    }
}
```  

<pre>
BenchmarkGoroutine        500000              3097 ns/op             164 B/op       11 allocs/op
BenchmarkSequential     10000000               224 ns/op               0 B/op        0 allocs/op
</pre>  
  

  
### for 루프를 돌리면서 조건식 부분에 직접 len() 을 써나 사전에 변수에 넣은 것을 써도 별로 차이 나지 않는다

```
package bench_test

import "testing"

func BenchmarkLenDirect(b *testing.B) {
    n := 1000
    s := make([]string, n)
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        for j := 0; j < len(s); j++ {
            _ = s[j]
        }
    }
}

func BenchmarkLenCached(b *testing.B) {
    n := 1000
    s := make([]string, n)
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        for j, length := 0, len(s); j < length; j++ {
            _ = s[j]
        }
    }
}
```  

<pre>
BenchmarkLenDirect       1000000              1221 ns/op               0 B/op        0 allocs/op
BenchmarkLenCached       1000000              1221 ns/op               0 B/op        0 allocs/op
</pre>  
  

  
## 리플렉션은 최후의 수단
퍼포먼스에 기여하지 않는 부분에서만 사용한다.  
어디가 퍼포먼스에 기여하는지가 불투명 하다면 사용 금지 쪽이 좋다.  
한번 쓰면 리플렉션은 다양하게 사용하고 싶어지는 마력이 있다.  
  
  
### 이 글에서 사용한 벤치 마크는
- https://github.com/naoina/go-adventcalendar-2014-bench  
  