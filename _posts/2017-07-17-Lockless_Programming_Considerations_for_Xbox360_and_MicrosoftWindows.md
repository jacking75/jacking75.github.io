---
layout: post
title: Lockless Programming Considerations for Xbox 360 and Microsoft Windows
published: true
categories: [threading]
tags: threading lock-less memory-barriers atomic
---
[Lockless Programming Considerations for Xbox 360 and Microsoft Windows](https://msdn.microsoft.com/ko-kr/library/ee418650(v=vs.85).aspx)  
라는 글의 결론  
  
### 플랫폼 별 동작 차이
* InterlockedXxx 함수에서 CPU에 의한 읽기/쓰기 순서 변경을 방지할 수 있는 것은 Windows로 한정된다. 이 함수를 Xbox 360에서 사용해도 CPU에 따른 순서 변경을 방지할 수 없다.  
* Visual Studio C++2005에서 volatile 변수를 사용한 경우 Windows에 한해 CPU에 의한 읽기/쓰기 순서 변경을 방지할 수 있다. Xbox 360에서는 컴파일러에 의한 읽기/쓰기 순서 변경 밖에 방지할 수 없다.
* Xbox 360에서는 쓰기 순서가 변경된다. x86 또는 x64에서는 쓰기 순서는 변경 되지 않는다.
* Xbox 360에서는 읽기 순서가 변경되지만, x86 또는 x64에서는 읽기 순서는 읽기와 쓰기 타깃 장소가 다른 때만 쓰기에 대해서 변경된다.
  
  
### 권장 사항
* 가능하면 보다 확실한 lock을 사용하자.
* lock을 자주 사용하는 것은 피하고 비용 증대를 막도록 하자다.
* 스톨이 발생하기 때문에 lock을 너무 오랫동안 유지하는 것은 피한다.
* 까다롭다는 결점을 보충하고도 남는 가치가 있는 경우에만 lock-less 프로그래밍을 사용하자.
* 수동적 프로시저 호출과 보통의 코드들 사이에서 데이터를 공유하는 경우 등 다른 lock이 금지된 상황에서 lock-less 프로그래밍 또는 스핀 lock을 사용하자.
* 올바르게 동작이 증명된 표준적인 lock-less 프로그래밍 알고리즘 이외는 이용하지 않도록 하자.
* lock-less 프로그래밍을 할 경우 필요에 따라서 volatile 플래그 변수와 메모리 장벽 명령을 사용한다.
* Xbox 360에서는 Acquire 버전과 Release 버전의 InterlockedXxx 함수를 이용할 수 있다.