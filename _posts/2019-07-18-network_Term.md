---
layout: post
title: 네트워크 용어
published: true
categories: [Network]
tags: 용어
---
## Receive-Side Scaling(RSS)                     
- http://minimonk.net/2728
- Windows Server 2012의 NIC 티밍(teaming)에 대응.
  
  
## TCP Chimney Offload
- http://vstarmanv.tistory.com/58
- VM에서는 사용 불가.
- Windows Server 2012의 NIC 티밍에 비 대응.
  
  
## NetDMA
- 네트웍 수신 데이터를 네트웍 어댑터의 수신 버퍼에서 애플리케이션 버퍼에 복사하는 작업을 CPU로 하지 않고 칩셋을 포함한 시스템 전체가 연대하여 DirectMemory Access에 대응하도록 하는 기술.
- Intel I/O Acceleration Technology(I/OAT).
- NetDMA와 같이 TCPChimney Offload를 사용할 수 없다. 하드웨어 지원 및 설정이 양쪽 모두 사용으로되어 있는 경우 TCP Chimney Offload를 우선 시 한다.
- Windows Server 2012에서는 사용 불가.
  
  
## TCP/IP Offload
- http://bugtruck.tistory.com/95
  
  
## Receive SegmentCoalescing(RSC)
- 대량의 네트웍 I/O 트래픽을 처리할 때 오버헤드를 줄여준다.
- 구체적으로는 복수의 수신 패킷을 모아서 하나의 패킷으로 만들어서 처리 횟수를 줄여준다.
  
  
## Jumbo Frame
- 패킷의 프레임 사이즈를 확대하여 데이터 전송 호율을 올린다.
- 이더넷 MTU 1500바이트에서 최대 9K까지 대응. GBit 이상의 네트웍에서 사용.
- 처리 오버헤드 감소.
- 플라그멘트화 감소.
- 인터럽트 처리 감소.
  
  
## InfiniBand
- InfiniBand는 2000년에업계 모임인 InfiniBand Trade Association에 의해서 책정된 규격.
- 슈퍼 컴퓨터 등 HPC(High Performance Computing)분야에서 사용되고 있는 서버간 데이터 통신 기술의 하나.
- 이더넷에 의한 LAN 접속과 비슷하게 InfiniBand도 수 십 미터 범위에서의 고속 데이터 통신에 사용되고 있다.그러나 통신 케이블의 광 케이블화에 의해서 통신 범위를 수 백 미터 단위까지 늘리고 있으면 2011년에는 112Gbps 데이터 전송을 하는 기기도 나와 있다. 현재에도 성능확장이 진행되고 있는 규격.

