---
layout: post
title: golang - 추첨 기능 라이브러리 lottery 
published: true
categories: [Golang]
tags: go lottery random 추첨
---
- [github](https://github.com/kyokomi/lottery)
- 추첨 기능 라이브러리
- random을 랩핑
  

## 지정한 확률
20%의 확률로 ~ 같은 처리를 하고 싶을 때    

```
lot : = lottery.New (rand.New (rand.NewSource (time.Now () UnixNano ())))

if lot.Lot ( 20 ) {
     // 20% 일 때의 처리
}
```  
  
    
## 추첨 목록에서 1건 추첨
다음과 같은 여러 추첨 대상에서 1건만 추첨 할 때의 interface도 있다.  
- A : 10 %
- B : 20 %
- C : 30 %
- D : 40 %  
  
Prob() int라는 인터페이스를 구현하면 OK  

```
type DropItem struct {
    ItemName string 
    DropProb int
}

func (d DropItem) Prob () int {
     return d.DropProb
}

var _ lottery.Interface = (* DropItem) ( nil )
```  

추첨은 이런식으로 한다  

```
lot : = lottery.New (rand.New (rand.NewSource (time.Now () UnixNano ())))

// 추첨 대상 목록
dropItems : = [] lottery.Interface {
    DropItem {ItemName : "에릭서" , DropProb : 10 },       // 10 % 
    DropItem {ItemName : "에테르" , DropProb : 20 },       // 20 % 
    DropItem {ItemName : "물약" , DropProb : 30 },     // 30 % 
    DropItem {ItemName : "꽝" , DropProb : 40 },        // 40 %
}

// 추첨
lotIdx : = lot.Lots (dropItems ...)
if lotIdx == - 1 {
     // error입니다 (추첨 목록 지정 실수)
}

// 추첨 결과 Item
dropItem : = dropItems [lotIdx] (DropItem)
```  
  
아래 테스트 코드도 참고로   
https://github.com/kyokomi/lottery/blob/2c0264f14a47bf1be830ae12d2c84a90ce9f1ffa/lottery_test.go  
  

  
## 예제
https://gist.github.com/tenntenn/2d0977d317222339cd2e  

```
package main

import (
	"fmt"
	"math/rand"
	"time"

	"github.com/kyokomi/lottery"
)

type Ball struct {
	Color string
	Count int
}

func (b *Ball) Prob() int {
	return b.Count
}

func (b *Ball) String() string {
	return fmt.Sprintf("%s(%d)", b.Color, b.Count)
}

func main() {
	lot := lottery.New(rand.New(rand.NewSource(time.Now().UnixNano())))
	red := &Ball{"Red", 1}
	green := &Ball{"Green", 10}
	blue := &Ball{"Blue", 3}

	bag := []lottery.Interface{red, green, blue}

	for {
		i := lot.Lots(bag...)
		if i < 0 {
			break
		}
		b, _ := bag[i].(*Ball)
		b.Count = b.Count - 1
		fmt.Println(b.Color, "--", red, green, blue)
	}
}
```  
결과  
<pre>
Blue -- Red(1) Green(10) Blue(2)
Green -- Red(1) Green(9) Blue(2)
Blue -- Red(1) Green(9) Blue(1)
Green -- Red(1) Green(8) Blue(1)
Green -- Red(1) Green(7) Blue(1)
Green -- Red(1) Green(6) Blue(1)
Green -- Red(1) Green(5) Blue(1)
Green -- Red(1) Green(4) Blue(1)
Green -- Red(1) Green(3) Blue(1)
Blue -- Red(1) Green(3) Blue(0)
Red -- Red(0) Green(3) Blue(0)
Green -- Red(0) Green(2) Blue(0)
Green -- Red(0) Green(1) Blue(0)
Green -- Red(0) Green(0) Blue(0)
</pre>  