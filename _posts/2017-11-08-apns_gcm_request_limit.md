---
layout: post
title: APNS/GCM의 리퀘스트 제한
published: true
categories: [Network]
tags: network apns gcm push mobile
--- 
번역 글로 2016/10월 기준. 이후 바뀔 수 있음  
   
## APNS와 GCM에서 Push 통지를 할 때의 제한 정리  
   
### APNS  
iOS8 미만의 경우 페이 로드 데이터의 상한은 256 바이트까지.  
iOS8의 경우 페이 로드 데이터의 상한은 2K 바이트까지.  
iOS9 이후의 경우 페이 로드 데이터의 상한은 4K 바이트까지.  
1회 통신에서 전 패킷이 5000~7000 바이트를 넘으면 APNS 에서 절단된다?(확실하지 않음)  
패킷의 제한은 상당히 완만하지만 너무 빠른 리퀘스트는 에러가 되므로 1리퀘스트당 10㎜~50밀리 초 정도의 간격을 두는 것이 좋을 듯(확실하지 않음)  
   
4번째와 5번째 정보는 비공식이고 상반되는 것이기 때문에 결국은 스스로 확인할 수밖에 없다고 생각하는데 "적어도 초당 9000 메시지는 송신할 수 있다" 라고 Apple의 공식 문서에 있는 것 같아서 일단 너무 한꺼번에 대용량의 메시지를 발송하지 않고 가끔 sleep 처리를 넣으면 괜찮지 않을까 생각한다.  
    
<br>  
   
### GCM 
페이 로드 데이터의 상한은 4096 바이트까지. 
멀티 캐스트 전송의 경우 최대 1000 단말까지.  
   
모두 공식 정보이므로 확실함.  
    
<br>  
   
### 어떻게 해야 할까?  
위의 제한이 있으므로 멀티 OS 대응의 경우 서버 애플리케이션은 Push 메시지는 140 문자 이내(Push에서 그렇게 긴 메시지 보내지는 않을 것 ⇒ Twitter와 같은 글자 수만 있으면 충분하지! 라는 생각으로)로 하고, 수 천대 이상에 한번에 보내고 싶다면 GCM에 맞추어 1000건씩 분할, 동시에 APNS를 배려하여 각 처리 사이에서 0.1초 정도 간격을 주고, APNS 및 GCM에 요청하면 거의 괜찮지 않을까 생각한다.  
      
<br>  
<br>     
  
출처: http://qiita.com/itosho/items/f4ec3495c2d2782c5359
  