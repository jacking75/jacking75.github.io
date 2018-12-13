---
layout: post
title: golang - 주기적으로 실행하는 패턴
published: true
categories: [Golang]
tags: go timer
---
main에서는 아래와 같이 ctx를 만들어서 periodicLoop에 넘기고, 멈추고 싶을 때는 cancel()을 호출한다.  
  
```  
func main() {
    ...

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    go periodicLoop(ctx, 2*time.Second)

    /* call cancel() to stop */
}
```
  
```  
func doPeriodically(t time.Time) {
    /* do something */
}

func periodicLoop(ctx context.Context, interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()
    doPeriodically(time.Now())
    for {
        select {
        case <-ctx.Done():
            return
        case t := <-ticker.C:
            doPeriodically(t)
        }
    }
}
```