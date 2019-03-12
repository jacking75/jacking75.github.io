---
layout: post
title: C# - http 요청 시 동시 접속 수 제한
published: true
categories: [.NET]
tags: c# net
---
- 닷넷프레임워크는 서버로의 동시 접속 수가 기본으로 최대 2로 제한 되어 있다
    - 즉 클라이언트는 한 서버로 머신당 최대 동시 접속 수가 2개로 제한
- 그래서 웹 서비스 클라이언에서 웹 서비스 등을 호출할 때 출력결과를 올리기 위해서는 멀티스레드로 웹서비스를 호출하여도 실제로는 활동하는 최대 동시 접속 수는 2가 된다.
- 이 제한은 애플리케이션 구성 파일(machine.congif에서도 가능)에 connectionManagement 요소를 아래와 같이 늘리면 된다.
  
```
<system.net>
  <connectionManagement>
    <add address="*" maxconnection="20"/>
  </connectionManagement>
</system.net>
```
  
