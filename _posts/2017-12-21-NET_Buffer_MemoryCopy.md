---
layout: post
title: .NET - Buffer.MemoryCopy 함수
published: true
categories: [.NET]
tags: .net c# .netcore MemoryCopy
---
메모리 복사 속도를 올리기 위해서는 Array.Copy 보다는 Buffer.BlockCopy를 사용하는 것이 좋다.  
  
단 Buffer.BlockCopy는 작은 크기(10~30 바이트 이하)의 복사에서는 Array.Copy 보다 뛰어나다고 할 수 없다. 
원래 Buffer.BlockCopy에는 낭비가 있다.  
Buffer.BlockCopy는 네이티브의 C++ 코드를 호출하고, 타입 체크와 범용적인 타입에 의한 처리가 있다.  
  
Buffer.BlockCopy을 더 최적화 한 것이 NET 4.6에서 추가된 Buffer.MemoryCopy이다.  
SSE2를 사용할 수 있는 환경이라면 64바이트 단위(RyuJIT가 그렇게 한다), 아니면 8바이트 단위로 C#의 unsafe에서 복사한다.   
    
[MSDN Buffer.MemoryCopy 메서드](https://msdn.microsoft.com/ko-kr/library/dn823273(v=vs.110).aspx)