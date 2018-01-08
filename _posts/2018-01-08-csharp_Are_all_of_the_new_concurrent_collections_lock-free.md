---
layout: post
title: .NET의 모든 병렬 컨테이너는 lock-free 인가?
published: true
categories: [.NET]
tags: c# .net concurrent collections lock-free
---
[FAQ :: Are all of the new concurrent collections lock-free?](https://blogs.msdn.microsoft.com/pfxteam/2010/01/26/faq-are-all-of-the-new-concurrent-collections-lock-free/)  
의 글을 일부 번역/정리.  
  
(이 대답은 .NET Framework 4를 기반으로 한다. 아래의 세부 정보는 문서화 되지 않은 구현 세부 사항이므로 향후 릴리스에서 변경 될 수 있다.)  
  
새로운 System.Collections.Concurrent 네임 스페이스의 모든 컬렉션은 보통 성능 이점을 얻기 위해 어느 정도 lock-free 기술을 사용하지만 일부는 기존 잠금이 사용된다.  
  
lock-free 기술에 의존하는 것이 때로는 가장 효율적인 해결책이 아닐 수 있다는 것에 주의해야 한다.  
우리가 "lock-free"라고 말하면 잠금(.NET에서의 전통적인 상호 배제 잠금은 일반적으로 C#의 "lock" 키워드 또는 Visual Basic "SyncLock" 키워드를 통해  System.Threading.Monitor 클래스를 사용할 수 있다)을 memory barriers와 비교 및 ​​스왑 CPU 명령을 사용하여 피할 수 있다(.NET에서는 "CAS" 작업을 System.Threading.Interlocked 클래스를 통해 사용할 수 있다).  
  
  
ConcurrentQueue<T> 및 ConcurrentStack<T>은 이러한 방식으로 완전히 lock-free 이다.  
이들은 결코 lock을 사용하지 않는다. 그러나 이들은 spin과 CAS 작업이 실패하여 경쟁에 직면했을 때 작업을 다시 시도 할 수 있다.  
  
ConcurrentBag<T>은 동기화 필요성을 최소화 하기 위해 다양한 메커니즘을 사용한다.  
예를 들어, 접근하는 각 스레드에 대해 로컬 큐를 유지 관리하며 일부 조건에서는 스레드가 거의 또는 전혀 경합하지 않아서 lock-free 방식으로 로컬 큐에 접근 할 수 있다.  
따라서 ConcurrentBag<T>는 때때로 lock을 요구하지만 특정 동시 시나리오(예 : 많은 스레드가 동일한 속도로 생성 및 소비하는 경우)에 매우 효율적인 콜렉션이다.  
  
ConcurrentDictionary<TKey, TValue>는 Dictionary에 데이터를 추가하거나 업데이트 할 때 세밀한 lock을 사용하지만 읽기 작업에는 lock-free를 사용한다.  
그래서 Dictionary에서 읽는 것이 가장 빈번한 작업 인 시나리오에 최적화되어 있다.  
  