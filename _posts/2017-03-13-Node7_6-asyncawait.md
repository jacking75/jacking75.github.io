---
layout: post
title: Node 7.6 에서 async/await를 기본으로 지원
published: true
categories: [Node.js]
tags: Node.js 번역 async/await
---
Node.js 7.6 이 출시 되었다.  
async/await 지원이 기본적으로 유효하게 되어 낮은 메모리 디바이스에서의 성능이 개선되었다.  
  
Node 7.6의 async/await 지원은 Chromium의 JavaScript 엔진인 V8를 버전 5.5로 업데이트한 데 따른 것이다. 이것이 의미하는 것은 async/await는 이제 실험적인 것이 아니라는 점이다.  
  
async/await의 가장 큰 장점은 비동기 오퍼레이션의 시퀀스를 콜백을 통해서 중첩하는 "콜백 지옥"을 회피할 수 있는 것이다.

예를 들어, 콜백을 이용한 2개의 비동기 연산 처리는 아래와 같다.  
  
```
function asyncOperation(callback) {
  asyncStep1(function(response1) {
    asyncStep2(response1, function(response2) {
        callback(...);
    });
  });
}
```
  
async/await를 사용하면 코드가 간소화되어, 마치 동기 오퍼레이션 시퀸스처럼 보인다.

```
function asyncOperation() {
  return asyncStep1(...)
    .then(asyncStep2(...));
}
```  
  
Node 7.6 에서는 V8 5.5외에 크로스 플랫폼 비동기 I/O 라이브러리 libuv가 1.11로 zlib이 1.2.11으로 등 의존 관계도 업데이트 되고 있다.
  
 
<br>
<br>  

출처: https://www.infoq.com/news/2017/02/node-76-async-await 