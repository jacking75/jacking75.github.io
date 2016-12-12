---
layout: post
title: socket의 listen의 backlog 수와 Accept
published: false
categories: [Network Programming]
tags: Network_Programming socket
---

네트웍 프로그래밍에서 서버가 되는 애플리케이션은 socket을 바인딩한 후 listen()을 해야 접속을 받을 수 있습니다.

이 때 listen() 함수에는 파라메터로 backlog라는 숫자를 설정합니다.

IOCP를 사용하여 네트웍 프로그래밍을 할 때는 보통 미리 Accept를 걸어 놓고, 이후 클라이언트에서 접속을하면 미리 Accept를 걸어 놓은 socket을 할당합니다.



여기서 고민이 생기는 것이 backlog 수를 얼마를 줘야 될지가 의문이 생길 수 있습니다. 특히 IOCP를 사용할 때는 접속을 허용할 숫자만큼만 미리 Accept를 걸어 놓으므로 이것을 다 사용하며 클라이언트가 접속을 시도하면 backlog가 허용하는 수만큼 접속은 성공하지만 서버는 이 연결을 인지하지 못합니다. 클라이언트가 계속 연결을 끊지 않고 있는 상태로 있다가 서버에서 새로운 Accept를 걸면 그 때서야 서버는 새로운 연결이 있음을 알아차립니다.

( IOCP에서 미리 Accept를 100개 걸어 놓고, backlog 수가 10이라면 110개의 연결은 성공합니다. 다만 100개는 서버와 완전 연결이 되는 것이고 나머지 10개는 대기 상태에 놓이게 됩니다 )



허용된 연결만 서버에서만 받고 싶다면 두 가지 방법이 있습니다.

1.Socket 옵션인 SO_CONDITIONAL_ACCEPT 사용 – TCP에서 관리

2.서버에서 허용할 연결 + backlog 수만큼 미리 Accept를 걸어 놓고 허용 수를 넘은 연결은 짜른다. – 애플리케이션에서 관리



첫 번째 방법을 사용하여 연결을 허용할 숫자를 설정하면 시스템 단에서 클라이언트의 연결 요청을 거부합니다. 간단하게 허용할 연결 숫자를 관리할 수 있습니다.

하지만 이 방법은 단점이 있습니다.

1. TCP 스택에서 지원하는 SNK 공격 방어 기능이 비 활성되어 이 기능을 애플리케이션에서 처리해야 합니다.

2.Accept 완료가 느려지면 연결을 허용할 수 있는 상황에서도 연결 실패가 일어납니다.

이 이외에도 몇몇 단점이 있기 때문에 웹에서 본 문서 중에서 사용하지 않기를 바라는 사람도 있었습니다.



그래서 건즈2에서는 두 번째 방법을 사용하기로 했습니다.



MSDN http://msdn.microsoft.com/en-us/library/dd264794(VS.85).aspx

SO_CONDITIONAL_ACCEPT 설명 및 단점 지점 http://www.codeguru.com/forum/archive/index.php/t-323371.html



구글에서 찾은 SO_CONDITIONAL_ACCEPT 관련 한글 문서

http://mirine35.egloos.com/5057698

http://whiteberry.egloos.com/1848774

http://symlink.tistory.com/52

http://symlink.tistory.com/37

http://cafe.naver.com/ongameserver/2263

http://cafe.naver.com/ongameserver/2338

http://cafe.naver.com/ongameserver/3043
