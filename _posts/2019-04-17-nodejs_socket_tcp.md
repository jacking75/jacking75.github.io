---
layout: post
title: Node.js - TCP Socket
published: true
categories: [Node.js]
tags: c# Node.js socket tcp
---
## 간단한 hello world
Class: net.Server 이벤트  
<pre>
'listening': server.listen()를 호출 후 서버가 바인드 되었을 때 발생.
'connection': 새로운 연결이 만들어지면 발생.
'close': 서버가 끊어지면 발생.
'error': 에러 발생. 이 이벤트 이후 'close' 이벤트가 직접 생성되는 경우도 있다.
</pre>
  
  
Class: net.Socket 이벤트  
<pre>
'connect': 소켓 커넥션 확립이 성공했을 때 발생.
'data': 데이터를 받으면 발생.
'end': 상대방이 FIN 패킷을 보냈을 때 발생.
'timeout': 소켓이 타임아웃 하여 비활성화된 경우에 발생.
'drain': 쓰기 버퍼가 빈 경우에 발생.
'error': 에러가 발생했을 때. 'close' 이벤트는 이 이벤트 후에 직접 호출된다.
'close': 소켓이 완전하게 끊어진 경우에 생성된다. 인수 had_error는 boolean으로 소켓이 전송 에러로 끊어졌는지 나타낸다.
</pre>
  
  
접속한 클라이언트에 지정된 메시지를 보낸다.  
```
net = require('net');

var s = net.createServer();
s.addListener('connection', function (c) {
    c.write('[tcp/ip] hello world!\n');
});

console.log('start tcp~~~');

s.listen(23452);
```
  
  
socket 에 이벤트 등록  
```
var s = net.createServer( function(socket) {
  socket.on('data', function(data)) {
    .....
  });

  socket.on('end', function() {
    .....
  });

});
```
  
  
  
### 간단한 클라어언트
  
```
var net = require('net');

var client = new net.Socket();
client.setEncoding('utf8');

client.connect('3000', 'localhost', function(){
  console.log('client-> connected to server');
  client.write('Who needs a browser to communicate?');
});

process.stdin.resume();

process.stdin.on('data', function(data){
  client.write(data);
});

client.on('data', function(data){
  console.log('client-> ' + data);
});

client.on('close', function(){
  console.log('client-> connection is closed');
});
```
  
  
  
### 접속한 클라이언트 관리
출처: http://egloos.zum.com/spectrum/v/5577613  

```
var clients = [];
require('net').createServer(function(socket) {
  socket.on("connect", function() {
    clients.push(socket);
  });
  socket.on("close", function() {
    clients.splice(clients.indexOf(socket), 1);
  });
  socket.on("data", function(data) {
    var sender = this;
    clients.forEach(function(client) {
      if (client !== sender) client.write(data);
    });
  });
}).listen(1337,"0.0.0.0");
```
  
  
  
### 접속 후 통신하지 않는 클라이언트  연결 종료하기
idle socket은 setTimeout(milliseconds)로 traffic 이 없을 경우 connection을 종료할 수 있다.  
```
var timeout=6000;
socket.setTimeout(timeout);
socket.on('timeout',function(){
socket.write('idle timeout..');
socket.end();
});
```
  
또는 다음과 같이 줄일 수 있다.
  
```
socket.setTimeout(6000,function(){
socket.end('idle time, disconnecting..');
});
```
  
  
  
### keep-alive 설정
ACK flag를 가진 빈 TCP Packet을 보내서 connection을 유지할 수 있다.
  
```
socket.setKeepAlive(true);
```
  
packet을 보내는 주기를 지정 가능하다
  
```
socket.setKeepAlive(true,10000);
```
  
  
  
### TCP Buffer의 전송 Delay 설정
socket.setNoDelay(true); 로 write시 데이터를 즉시 보내게 할 수 있다. (socket.setNoDelay(false); 로 해제)
  
  
  
### listening client connections
host가 생략되면 모든 IP Address에서 들어온 connection을 받아들인다.
  
```
var port=4001;
var host='127.0.0.1';
server.listen(port,host);
```
  
  
  
### closing server
  
```
server.close()
server.on('close',function(){
console.log('server closed');
});
```
  
  
  
### socket.setEncoding('utf8')
TCP Socket의 setEncoding() 메서드를 수행하면, 'data' Event 발생시 callback으로 들어오는 데이터(data), 즉 수신 데이터의 타입이 String으로 변경된다.  
바이너리 데이터를 주고 받을 때는 'setEncoding()'를 사용하지 않아야 한다.  
  
  
  
### 바이너리 데이터 파싱하기
C++ 에서는 아래의 데이터를 보낸다.  

```
struct CTestData {
 char a;
 int b;
 char c[15];
};
```
  
node.js쪽  
```
client.on('data', function(data) {
 chara = data.slice(0, 4);
 intb = data.slice(4, 8);
 charc = data.slice(8, 24);

 console.log('char a :', chara.toString('utf-8'));
 console.log('int b :', intb.readUInt8());
 console.log('char c :', charc.toString('utf-8'));
```
  
  
  
### Buffer를 사용하여 바이너리 데이터 보내기
  
```
var net = require('net');

var buffer = new Buffer(3);
buffer.writeUInt8(0x01, 0);
buffer.writeUInt8(0x02, 1);
buffer.writeUInt8(0x03, 2);

var client = net.connect(8000, "127.0.0.1", function() {
    client.write(buffer);
});
```
  
  
  
### 참고
- [v0.6.21 기준 .net 문서](http://nodejs.jp/nodejs.org_ja/docs/v0.6/api/net.html)
- [Node.js 에서 구현하는 소켓 서비스](https://mylko72.gitbooks.io/node-js/content/chapter8/intro.html)
- [Node TCP 서버 클라이언트](http://takaheraw.hatenablog.com/entry/2013/01/04/115434)
- [node.js chat server with QT client](http://jeremyko.blogspot.kr/2012/12/nodejs-tcp-chatting-server-with-qt.html?m=0)
    - [nodeChatServer](https://github.com/jeremyko/nodeChatServer)
