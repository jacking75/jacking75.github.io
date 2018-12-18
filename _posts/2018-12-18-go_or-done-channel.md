---
layout: post
title: golang - or-done-channel로 코드 가독성을 높인다
published: true
categories: [Golang]
tags: go channel done
---
[원문](https://ymotongpoo.hatenablog.com/entry/2017/12/04/091403)  
  
## Go 병렬 패턴
- for-select loop
- or-channel
- or-done-channel
- tee-channel
- fan-in, fan-out
등이 있다. 
  
     
## 어떤 때 or-done-channel을 사용하나
상한 수가 정해져 있는 것 같은 처리를 할 경우  
- 데이터 소스에서 입력이 끝나 버린 경우(channel의 close에 대응)
- 최대 수에 도달 한 경우(done에 대응)
중 하나에 해당하는 경우 유용한 패턴이다.  
  

## range를 사용하는 경우
우선 channel이 close 될 때까지 처리를 할 경우는 range 키워드로 for 루프를 돌리는 것이 편리하고 가독성도 높다.  
```
for v : = range ch {
    something (v)
}
```  

그러나 이 경우 프로세스 한계에 도달 한 경우에 대응하지 못한다. 어떻게 든 ch와 done을 모두 받아 줄 필요가 있다.  
  
    
## for-select loop만을 사용하는 경우
이렇게 되면 Go에서 여러 channel을 사용한 병렬 처리를 쓰려고 하는 경우에는 for-select loop가 가장 자주 나오므로 이것을 쓰고 싶어진다.  

```
LOOP :
 for {
     select {
     case v, ok : = <-ch :
         if ! ok {
             break LOOP
        }
        something (v)
    case <-done :
         break LOOP
    }
}
```  
  
어떤 식으로든 원하는 처리는 쓸 수 있지만, 아무래도 외형이 커져버린다.  
그래서 이 과정을 별도로 정리하여, range 작성에 넣는 것이 or-done-pattern 이다.

    
## or-done-channel
defer의 성질을 이용하여 done이 와도 ch에서의 데이터 취득이 끝난 경우에도 반드시 stream을 close하고 있다.  
이것으로 얻어진 stream이 닫혔는지 어떤지만 의식하면 잘 되기 때문에 보통 range 패턴에 넣는 것이다.  

```
func orDone (done <- chan  bool , ch <- chan Data) <- chan Data {
    stream : = make ( chan Data)
     go  func () {
         defer  close (stream)
         for {
             select {
             case <-done :
                 return 
            case v, ok : = <-ch :
                 if ! ok {
                     return
                }
                stream <- v
            }
        }
    } ()
    return stream
}

for v : = range orDone (done, ch) {
    something (v)
}
```  
