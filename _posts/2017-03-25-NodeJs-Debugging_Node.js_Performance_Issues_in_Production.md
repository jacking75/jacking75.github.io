---
layout: post
title: Debugging Node.js Performance Issues in Production
published: true
categories: [Node.js]
tags: Node.js 번역 performance
---
아래 강연의 내용의 일부를 정리  
[Debugging Node.js Performance Issues in Production](https://www.youtube.com/watch?v=_LkafZJ8TD4)  
  
  
### 느리게 되는 이유

- single thread
- CPU intensive code
- Slow I/O
- Event Loop saturation 
- Running out of memory
- Garbage Collection


#### CPU intensive code examples:
- Sync I/O `fs.*Sync`
- JSON.parse
- RegExp
- Crypto
- Templates


### 0x
https://www.npmjs.com/package/0x  
0x는 Linux와 OS X 를 지원하며 단일 명령어로 Node 프로세스의 인터렉티브한 프레임 그래프를 생성한다.   
node.js 프로세스의 스택 트레이스를 비주얼하게 볼 수 있어서 성능 조사 때 도움이 된다.  

