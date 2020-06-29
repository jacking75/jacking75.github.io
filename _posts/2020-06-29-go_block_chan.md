---
layout: post
title: golang - chan에 보내기를 할 때 버퍼 공간 부족에 의한 블럭킹 여부 판별
published: true
categories: [Golang]
tags: go chan channel
---
chan의 공간이 다 찰 때까지 보내기를 하면 빈 공간이 생길 때까지 블럭킹 상태가 된다.  
블럭킹 상태에 빠지지 않게 하는 방법 중의 하나의 `select`와 조합해서 사용한다.  
  
```
ch1  chan []byte

ch1 = make(chan []byte, config.MaxChannelSize)

func sendData(data []byte) {
  select {
      case ch1 <- data:
      {
        return
      }
      default: // ch1이 블럭킹 상태가 되면 default가 호출된다.
      {
        //여기가 호출되면 ch1 이 꽉 찬 상태이다.
      }
  }
}
```  
  
