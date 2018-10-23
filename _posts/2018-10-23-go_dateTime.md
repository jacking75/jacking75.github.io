---
layout: post
title: golang - time
published: true
categories: [Golang]
tags: go time golang
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
# 시간 및 날짜
[표준 API](http://golang.org/pkg/time/)  
  
  
## 표준 라이브러리 사용 예
### 현재 날짜

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now)
}
```

  
### 현재의 연, 월, 일, 분, 초

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.Year())
    fmt.Println(now.Month()
    fmt.Println(now.Day())
    fmt.Println(now.Hour())
    fmt.Println(now.Minute())
    fmt.Println(now.Second())
}
```

  
### 현재 요일

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.Weekday())
}
```

  
### 1초 후

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.Add(time.Duration(1) * time.Second))
}
```

  
### 1분 후

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.Add(time.Duration(1) * time.Minute))
}
```

  
### 1시간 후

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.Add(time.Duration(1) * time.Hour))
}
```

  
### 하루 뒤

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.Add(time.Duration(24) * time.Hour))
}
```
혹은  
```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.AddDate(0, 0, 1))
}
```

  
### 1개월 후

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.AddDate(0, 1, 0))
}
```

  

### 1년 후

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now.AddDate(1, 0, 0))
}
```

  
### 지난 달 첫날/말일
첫날은 그대로인데 말일은 다음달의 첫날을 얻고 거기서 하루 돌아가는 방식. 다른 언어에서도 이런식으로  
```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    firstDay := time.Date(now.Year(), now.Month()-1, 1, 0, 0, 0, 0, now.Location())
    lastDay := time.Date(now.Year(), now.Month(), 1, 0, 0, 0, 0, now.Location()).AddDate(0, 0, -1)
    fmt.Println(firstDay)
    fmt.Println(lastDay)
}
```

    
### 이달 첫날/말일

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    firstDay := time.Date(now.Year(), now.Month(), 1, 0, 0, 0, 0, now.Location())
    lastDay := time.Date(now.Year(), now.Month()+1, 1, 0, 0, 0, 0, now.Location()).AddDate(0, 0, -1)
    fmt.Println(firstDay)
    fmt.Println(lastDay)
}
```

  
### 다음 달 첫날/말일

```
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    firstDay := time.Date(now.Year(), now.Month()+1, 1, 0, 0, 0, 0, now.Location())
    lastDay := time.Date(now.Year(), now.Month()+2, 1, 0, 0, 0, 0, now.Location()).AddDate(0, 0, -1)
    fmt.Println(firstDay)
    fmt.Println(lastDay)
}
```

  
### Time 형의 값이 24시간 전인지 조사
24시간 경과하고 있거나 30일 경과하고 있었는지 등 Since 사용하면 좋음.  
```
package main

import (
    "fmt"
    "time"
)

func main() {
    target := time.Now().AddDate(0, 0, -1)
    expireDate := time.Duration(24) * time.Hour
    fmt.Println(time.Since(target) >= expireDate) //=>true
}
```

  
### Time 형의 값이 지정한 기간 내에 포함되는지
After와 Before를 조합하면 좋다.  
```
package main

import (
    "fmt"
    "time"
)

func main() {
    target := time.Now()
    tomorrow := target.AddDate(0, 0, 1)
    yesterday := target.AddDate(0, 0, -1)
    fmt.Println(target.After(tomorrow))   //=> false
    fmt.Println(target.After(yesterday))  //=> true
    fmt.Println(target.Before(tomorrow))  //=> true
    fmt.Println(target.Before(yesterday)) //=> false
}
```

    
### 날짜로부터 취득한다

```
fmt.Println(time.Date(2014, time.December, 31, 12, 13, 24, 0, time.UTC))
```  

<pre>
// Output:
// 2014-12-31 12:13:24 +0000 UTC
</pre>  
http://play.golang.org/p/iRfCerEQ8R  

월 Month에서 정의되어 있다. 또 인수의 마지막 location은 time.LoadLocation() 등에서 얻을 수 있다.  
```
loc, _ := time.LoadLocation("Asia/Tokyo")
fmt.Println(time.Date(2014, 12, 31, 8, 4, 18, 0, loc))
// Output:
// 2014-12-31 08:04:18 +0900 JST
```
http://play.golang.org/p/vQIZPA_1F3

  
### 문자열에서 얻는다

```
t, _ := time.Parse("2006-01-02", "2014-12-31")
fmt.Println(t)
// Output:
// 2014-12-31 00:00:00 +0000 UTC
```
http://play.golang.org/p/EAw_EloBxc  
  
- time.Parse() 첫째 인수는 Mon Jan 2 15:04:05 -0700 MST 2006에서 뽑는다.
```
t, _ := time.Parse("2006-01-02 15:04:05 MST", "2014-12-31 12:31:24 JST")
fmt.Println(t)
// Output:
// 2014-12-31 12:31:24 +0900 JST
```

    
### Unix 타임 시간 얻기

```
var curTime int64 = time.Now().Unix()
```  
  
      
### Unix TimeStamp에서 변환

```
fmt.Println(time.Unix(1419933529, 0))
// Output:
// 2014-12-30 09:58:49 +0000 UTC
```
http://play.golang.org/p/JtGB0MLKyx

제 2인수에서 나노 초를 지정할 수 있다.

  
### 시각 편집
시각의 조작과 차분을 확인할 경우 Duration 형을 다룬다. 내용은 int64.  
```
type Duration int64
```
  
명시적으로 Duration 형을 사용함으로써 시간 조작을 하고 있음을 알기 쉽게 한다.  
```
fmt.Println(reflect.TypeOf(1))               // int
fmt.Println(reflect.TypeOf(1 * time.Second)) // time.Duration
fmt.Println(reflect.TypeOf(1 * time.Hour))   // time.Duration
```
http://play.golang.org/p/TuxwkzdBkJ  

  
### ○ ○ 후의 시각을 얻는다

```
fmt.Println(t) // 2014-12-20 00:00:00 +0900 JST

t2 := t.Add(1 * time.Minute) //1분 후
fmt.Println(t2)              // 2014-12-20 00:01:00 +0900 JST

t3 := t.Add(1 * time.Hour) //1시간 후
fmt.Println(t3)            // 2014-12-20 01:00:00 +0900 JST

t4 := t.Add(24 * time.Hour) //하루 뒤(time.Day는 표준에는 없다)
fmt.Println(t4)             // 2014-12-21 00:00:00 +0900 JST
```
http://play.golang.org/p/vy_l5cJlHL  

하루 후의 경우는 time.AddDate()  
```
fmt.Println(t) // 2014-12-20 00:00:00 +0900 JST

t2 := t.AddDate(0, 0, 1) //하루 후
fmt.Println(t2)          // 2014-12-21 00:00:00 +0900 JST

t3 := t.AddDate(0, 1, 0) //1개월 후
fmt.Println(t3)          // 2015-01-20 00:00:00 +0900 JST

t4 := t.AddDate(1, 0, 0) //1년 후
fmt.Println(t4)          // 2015-12-20 00:00:00 +0900 JST
```
http://play.golang.org/p/iWF0DG-_St  

  
### ○ ○ 전의 시간을 얻는다
- time.Add() 에 - 를 넣는다

```
fmt.Println(t) // 2014-12-20 00:00:00 +0900 JST

t2 := t.Add(-time.Minute) //1분 전
fmt.Println(t2)           // 2014-12-19 23:59:00 +0900 JST

t3 := t.Add(-time.Hour) //1시간 전
fmt.Println(t3)         // 2014-12-19 23:00:00 +0900 JST

t4 := t.Add(-24 * time.Hour) //하루 전
fmt.Println(t4)              // 2014-12-19 00:00:00 +0900 JST
```
http://play.golang.org/p/5HcqtJN1o6   
  
- time.AddDate()에서 마이너스를 넣으면 과거가 된다  

```
fmt.Println(t) // 2014-12-20 00:00:00 +0900 JST

t2 := t.AddDate(0, 0, -1) //하루 전
fmt.Println(t2)           // 2014-12-19 00:00:00 +0900 JST

t3 := t.AddDate(0, -1, 0) //1개월 전
fmt.Println(t3)           // 2014-11-20 00:00:00 +0900 JST

t4 := t.AddDate(-1, 0, 0) //1년 전
fmt.Println(t4)           // 2013-12-20 00:00:00 +0900 JST
```
http://play.golang.org/p/Vki9YrIwWU    

      
### 2개의 시각을 비교한다  

```
t1 := time.Date(2014, 12, 20, 12, 0, 0, 0, loc)
fmt.Println(t1) // 2014-12-20 00:00:00 +0900 JST

t2 := time.Date(2014, 12, 20, 0, 0, 0, 0, loc)
fmt.Println(t2) // 2014-12-20 00:00:00 +0900 JST

fmt.Println(t1.Sub(t2)) // 12h0m0s
```
http://play.golang.org/p/dbaEeuxHof  

    
### 현재 시간에서 얼마나 앞인지 조사    

```
loc, _ := time.LoadLocation("Asia/Tokyo")

t1 := time.Date(2009, 11, 8, 8, 0, 0, 0, loc)
d := time.Since(t1)
fmt.Println(d) // 72h0m0s(Go Playground 에서 실행한 경우)
```
http://play.golang.org/p/X8ruti5dvc  
  
- time.Since(t)는 time.Now().Sub(t) 의 생략형.  
    
    
## Monotonic Clocks
Go1.8 이전 취득한 시각은 "wall clock"이라는 현재의 올바른 시각을 알기에 사용한다.  
한편 "monotonic clock"은 시간을 재기 위해서 쓰는 것이다.  
Go1.9부터는 time.Now에서 취득할 수 있는 시각에 "wall clock"과 "monotonic clock"이 포함된다.  
  
```Go
t := time.Now()
... operation that takes 20 milliseconds ...
u := time.Now()
elapsed := t.Sub(u)
```
위의 코드로 elapsed는 20ms가 되겠지만 실제로는 그렇게는 안 되는 케이스가 있다.  
구체적으로는 다음과 같은 사례이다.  
- ntpd 등으로 OS의 시각이 변경된 경우
- 윤초가 삽입·삭제된 경우  
  
Monotonic Clocks을 사용하는 위의 문제를 해결할 수 있다.  
[(일어)Go1.9에서 사용할 수 있는 Monotonic Clocks을 사용해 보았다](https://shogo82148.github.io/blog/2017/06/26/go19-monotonic-clock/)  