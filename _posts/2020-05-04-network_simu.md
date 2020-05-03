---
layout: post
title: 네트워크 상태 시뮬레이션 툴
published: true
categories: [Network]
tags: network simulating comcast clumsy tool
---
## comcast 
Simulating shitty network connections so you can build better systems.  
  
- https://github.com/tylertreat/comcast
- 네트워크 상태 시뮬레이션 툴이다.
- 패킷 드럅율이나 레이턴시를 임의로 조정할 수 있어서 네트워크 상태가 좋지 않은 경우에 애플리케이션이 문제 없이 잘 동작하는지 테스트 할 때 사용하면 좋다.
- Golang으로 만들었고, 내부적으로 iptables을 조작해서 시뮬레이션 상황을 만드는 것 같다.
- Linux, OSX를 지원하고 Windows는 지원하지 않는다(Windows의 경우 이런 기능을 하는 툴이 MS에서 제공하는 것이 있다)
  
  
  
## clumsy 0.2 
- http://jagt.github.io/clumsy/
- Windows OS에서 패킷 드랍율이나 지연을 만들어 주는 네트워크 상태 시뮬레이션 툴
  
  
  
## Network Emulator for Windows Toolkit
- https://network-emulator-for-windows-toolkit.software.informer.com/2.1/
- 설명: http://gameqa.tistory.com/120