---
layout: post
title: .NET 오프 소스 - stateless 라이브러리
published: true
categories: [.NET]
tags: .net c# .netcore stateless
---
- 유한 상태 머신을 쉽게 구현해 주는 라이브러리.
- 닷넷 코어 지원.
- Export to DOT graph 지원으로 상태 머신 구조를 이미지로 출력한다. 
- async/await 지원.
- [공식](https://github.com/nblumhardt/stateless)   
    
  
```
var phoneCall = new StateMachine<State, Trigger>(State.OffHook);

phoneCall.Configure(State.OffHook)
    .Permit(Trigger.CallDialed, State.Ringing);
	
phoneCall.Configure(State.Ringing)
    .Permit(Trigger.HungUp, State.OffHook)
    .Permit(Trigger.CallConnected, State.Connected);
 
phoneCall.Configure(State.Connected)
    .OnEntry(() => StartCallTimer())
    .OnExit(() => StopCallTimer())
    .Permit(Trigger.LeftMessage, State.OffHook)
    .Permit(Trigger.HungUp, State.OffHook)
    .Permit(Trigger.PlacedOnHold, State.OnHold);

// ...

phoneCall.Fire(Trigger.CallDialled);
Assert.AreEqual(State.Ringing, phoneCall.State);
```
