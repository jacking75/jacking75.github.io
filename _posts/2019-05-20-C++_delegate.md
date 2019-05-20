---
layout: post
title: C++ - Delegate 라이브러리
published: true
categories: [C++]
tags: c++ Delegate
---
상호 참조를 할 경우, 상호 참조 되는 클래스 중 둘 중 하나가 변경될 경우 상호 컴파일이 일어난다  
-> 작은 프로젝트라면 모르겠지만 대규모 프로젝트에서는 작은 소스 수정 하나가 큰 짐이 될 수 있다  
그리고 설계 상 상호 참조를 하고 있다는 것은 좋지 않다.  
   
상호 참조를 하는 대신 함수를 대신 호출하는 C#의 Delegate 개념을 사용하는 것이 좋다  
  
C++의 함수 포인터로 구현 할 수 있는데, 문제는 일반 함수 포인터와 달리 객체지향에서 주로 사용하는 클래스의 멤버 함수 포인터는 좀 복잡하다.  
(크기도 플랫폼 별로 다르고 해당 인스턴스의 타입도 알아야 하고..)  
  
boost 라이브러리의 function, bind 함수 등으로 간단하게 접근 할 수 있다(C++11 이상의 표준으로도 가능하다)  
하지만 이 방식을 사용하면 힙 메모리 영역을 사용하기 때문에 성능이 좋지 않다  
  
여러 개발자들이 이를 개선하기 위해 다양한 Fast-Delegate를 개발했다.  
[Fast C++ Delegate: Boost.Function 'drop-in' replacement and multicast](http://www.codeproject.com/Articles/18389/Fast-C-Delegate-Boost-Function-drop-in-replacement)  
  
그 중에서 [Sergey의 FastDelegate](https://www.codeproject.com/Articles/1170503/The-Impossibly-Fast-Cplusplus-Delegates-Fixed) 최신 버전을 사용해 보았다.  
사용 방법은 위 글과 예제 코드를 참고하기 바란다.  
  
[예제코드](/exampe_codes/delegate.zip )  