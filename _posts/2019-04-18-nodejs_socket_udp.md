---
layout: post
title: Node.js - UDP Socket
published: true
categories: [Node.js]
tags: c# Node.js socket udp
---
## 데이터 수신
  
```
dgram.createSocket(type, [callback])
```
  
type 에는 udp4, udp6, unix_dgram 지정 가능.  
  
콜백은 message 이벤트가 발생했을 때의 내용을 기술.  
```
function (msg, rinfo) { }
```
  
버퍼(msg)와 송신자 정보와 데이터의 바이트 수를 표시하는 정보(rinfo)로 구성된다.
  
  
  
## 간단 예제
UDP로 온 문자열을 표시하는 프로그램. 수신한 데이터는 그냥 바이트 열이므로 toString을 사용하여 문자열(ascii 문자열)로 변환한다.  
```
var dgram = require('dgram');

sock = dgram.createSocket("udp4", function (msg, rinfo) {
  console.log('got message from '+ rinfo.address +':'+ rinfo.port);
  console.log('data len: '+ rinfo.size + " data: "+
              msg.toString('ascii', 0, rinfo.size));
});

sock.bind(8000, '0.0.0.0');
```
  
  
  
## UDP 송신
  
```
dgram.send(buf, offset, length, port, address, [callback])
```
  
DNS 오류 등이 일어났을 때용으로 콜백도 지정할 수 있다(생략 가능). 보내는 데이터는 buf에서 지정된 데이터로부터 offset, length로 위치를 지정할 수 있지만, 그냥 단순히 문자열을 생성하고 보내는 경우는 기본적으로 offset=0, length=buf.length를 지정하는 형태가 된다. 영상 등의 큰 데이터를 분할하여 보내는 경우 등에는 offset, length를 활용할 필요가 있다.  
  
또한 buf에 관해서는 콜백 블록이 실행 되기 전 시점에서 재사용해서는 안 된다.  
  
  
  
### 예제 코드
  
```
var dgram = require('dgram');

sock = dgram.createSocket("udp4", function (msg, rinfo) {
  console.log('got message from '+ rinfo.address +':'+ rinfo.port);
  console.log('data len: '+ rinfo.size + " data: "+
              msg.toString('ascii', 0, rinfo.size));

  var message = new Buffer("nantekottai.");
  sock.send(message, 0, message.length, rinfo.port, rinfo.address);
});

sock.bind(8000, '0.0.0.0');
```
  