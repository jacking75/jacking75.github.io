---
layout: post
title: C# - 메모리
published: true
categories: [.NET]
tags: c# .net stackalloc memory gc
---
## 스택할당
- stackalloc 키워드를 사용하면 지역 변수에 한해서 스택 할당을 할 수 있음
- 해당 함수를 벗어나면 메모리는 삭제된다.
- unsafe 기능으로 사용할 수 있다.
- MSDN 문서
    - http://msdn.microsoft.com/ko-kr/library/cx9s2sy4.aspx
    - http://msdn.microsoft.com/ko-kr/library/aa664785(v=vs.71).aspx
- C#에서 포인터 사용하는 방법
    - http://rkddlsghk98.blog.me/30160902176
  
  
  
## 동적 메모리 할당
- MSDN
    - http://msdn.microsoft.com/ko-kr/library/aa664786(v=vs.71).aspx
  
  
  
## 메모리 최적화
- 필요한 만큼의 데이터 타입을 사용
- 싱글톤 사용으로 초기 로딩 시간을 줄이고 매번 메모리 할당을 하지 않아도 되므로 메모리 절약
- 메모리 폴링으로 재사용하기
- 데이터 스트리밍으로 대량의 메모리를 할당하지 않는다
- 퍼포먼스 모니터와 CLR Profiler로 조사
- 원문: [http://msdn.microsoft.com/en-us/magazine/cc163856.aspx]
  
  
  
## 64비트에서 2기가 바이트 보다 큰 배열 할당하기
- MSDN: http://msdn.microsoft.com/ko-kr/library/hh285054.aspx
  
  
  
## 서버 GC 사용하기
- 논리 프로세서 마다 힙을 가진다.
- app.config에서 
    - <gcServer enabled=”true” />
- 자세한 설명 http://www.simpleisbest.net/post/2011/04/27/Using-Garbage-Collection-Modes.aspx
  
  
  
## .NET Garbage Collection
- SustainedLowLatency 모드를 사용하여 GC를 계속적으로 off 시킬 수 있다
- 조언
    - 너무 잦은 객체 할당을 피해라
    - 너무 많은 참조를 피해라
    - 너무 많은 2세데 객체들을 만들지 마라
    - Dispose Pattern을 사용하여 Finalizer 호출을 삼가해라. Finalizer 호출되어야 하는 객체는 항상 1세대로 승급한다.
- http://xxxq.tistory.com/95
  
  
  
## GC 이해와 CLR
- http://www.hoons.net/board/cshaptip/content/44651
  
  
  
## 닷넷 가비지 컬렉션 다시 보기
- http://www.simpleisbest.net/?tag=/Garbage-Collection
  
  
  
## Dispose Pattern
- http://www.simpleisbest.net/post/2011/08/22/Dispose-Pattern-Basic.aspx
- http://www.simpleisbest.net/post/2011/08/29/Dispose_Pattern_Advanced.aspx
  
  
  
## 효과적인 C# 메모리 관리 방법
- http://dosun99v.blogspot.kr/2013/01/c.html
  
  
  
## 가비지 컬렉션
- 호출빈도를 줄이자. 동적메모리 사용의 낭비 제거.
- 문자열 조합. operator + (아는 내용) ---> StringBuilder로 변경. 지역변수로 선언해서. 매번 그 부분을 반복 사용하게 변경하는 방식. ^^
- XML파싱때 의외로 속도가 빨라졌음.
  
  
  
## 메소드 안에서 new
- Vector3를 메소드에서 new로 생성하는 코드들. 변수가 가비지로 생성한다.
- 자주 호출되는 경우. 멤버변수로 선언해서 사용하자.
- 메소드 안에서만 사용되는 인스턴스.
- 구조체로 바꾸자. 스택에 생성되니까
  
  
  
## Boxing/Unboxing
- Value, Reference Type. 서로 변환에 따른 가비지 생성에 따른 부하 문제. 박싱/언박싱은 느리니까.
- 어디서 많이 발생되는가?
- Collection. ArrayList, 해쉬테이블 같은 것.
- 부하가 심함. 힙과 스택을 오가다 보니. 가비지가 생성되어서 느려진다. 매프레임마다 하면 재앙수준이 되겠다.
- ArrayList -> List<>로 바꿔 쓰자.
  
  
  
## 메모리가 어디선가 늘고 있어요
- 의도하지 않은 참조.
- 관리 되지 않는 참조.
- 캐릭터 매니저 <-> 인스턴스 생성.(캐릭터 객체) <-> 캐릭터 위치 표시 객체
- 삭제했는데… 캐릭터 객체를 삭제하지 않는 경우.
- 여러 곳에서 class Char 객체 참조가 늘어나서... 어딘가에서 삭제 했을 때 내가 미처 못했던 곳에서 삭제되지 않아 문제 발생.
   
  
  
## WeakReference 소개
- 참조 전용 객체라면 WeakReference사용 권장. 단순 참조 하는 경우엔 권장.
- 가비지 컬렉션에 의한 회수를 허용하면서 참조.
- 삭제된 참조 값은 null 변함.  (가비지가 회수를 할 때 / 어디선가 삭제 될 때….)
- 문법은  WeakReference 이름 =  new WeakReference( 객체 );
- Dispose패턴을 이용해보자. 객체 인스턴스의 기능 정지를 명시하세요.
- 가비지 컬렉션은 자동으로 수행.
- 더 이상 사용되지 않는 것을 명시.
- Dispose()안에서 명시.
  
  
  