---
layout: post
title: AWS - P2P에서 클라우드로의 전환 : For Honor와 Friday the 13th The Game이 어떻게 플레이어 경험을 향상 시켰는가 
published: true
categories: [Cloud]
tags: cloud aws
---
[원문](https://aws.amazon.com/jp/blogs/gametech/for-honor-friday-the-13th-the-game-move-from-p2p-to-the-cloud-to-improve-player-experience/)  
  
게임 개발자로서 출시에 이르기까지 열정을 잃지 않고 게임 개발과 팬 커뮤니티의 육성에 수 년의 세월 투자 되어 왔다고 생각한다.  
최고의 사용자 경험을 제공 할 수 있는 것에 대해서는 예를 들어 네트워크 연결시 플레이어의 대기 시간에 대한 백엔드의 선택 등 백엔드 인프라는 가장 관심이 없을 수도 있다.  
  
많은 게임 개발자가 피어 투 피어(P2P) 네트워크에서 멀티 플레이어 유형의 게임을 실현시키고 있다.  
개발 초기에는 일견 비용 효율적인 솔루션으로 보이지만, 우리는 많은 개발자가 P2P에서 클라우드 상에서 가동하는 전용 게임 서버로 전환하는 것을 보았다.  
여기에서는 Ubisoft나 IllFonic 같은 스튜디오의 For Honor이나 Friday the 13th The Game 팀이 왜 P2P를 포기하고 클라우드의 전용 게임 서버로 갈아 탔는지 그 이유 중 일부를 보았다고 생각한다.    
  
## 열악한 대기 시간과 연결성에서 탈피
P2P 네트워크에서 동일 영역에서 강력한 네트워크 연결이 있으면 낮은 레이턴시 경험을 플레이어에게 가져다 준다. 그러나 P2P 네트워크에서의 전체적인 게임의 지연 시간은 호스트의 네트워크 연결 지연에 묶여 있다.  
  
For Honor는 당초 P2P 네트워크 모델을 채용하고 출시 되었다.  
시간이 지남에 따라 플레이어의 경험에 대해 아키텍처의 선택으로 인해 일부 문제가 있는 것이 밝혀져 세션의 이양 및 NAT(Network Address Transaction) 문제 해결이나  매치매이킹과 온라인 경험의 안정성울 얻기 위해 Ubisoft는 Amazon GameLift에서 실행하는 전용 게임 서버 모델로 전환 할 것을 결정했다.  
  
전용 게임 서버는 개발자 측에 의한 섬세한 제어를 가능하게 한다. 또한 클라우드에서 호스팅을 이용하는 것으로 플레이어를 낮은 레이턴시로 안정된 게임 서버에 연결하는 등의 관리도 쉽게 할 수 있다.  
  
  
## 드롭 아웃, 호스트의 우위성, 멀티 플레이어 게임의 치트 대책
플레이어는 네트워크 연결 문제, 게임에 대한 흥미 상실 등등 다양한 이유로 게임 세션에서 탈락한다.  
  
호스트 플레이어가 게임에서 빠져 나오면 다른 모든 플레이어가 게임 중단이나 순단을 초래한다.  
Friday the 13th The Game 독자의 독특한 비대칭 멀티 플레이 게임에서는이 문제가 더 현저했다.  
이 게임 내에서 그 악명 높은 제이슨은 매치 중의 다른 모든 플레이어와 적대한다.  
호스트 플레이어가 제이슨에 쓰러진 경우에 무덤에서 관전은 바라지 않고 게임에서 탈락 해 버리므로 모든 플레이어의 게임을 중단 해 버리고 있었다.  
  
P2P 네트워크에서는 부정이나 치트도 쉽게 가능하고, 호스트는 대기 시간으로 우위가 되는 것이 많이 있었다. 또한 전용 서버 모델 아키텍처에 비해 호스트 플레이어의 PC가 인증하는 P2P 모델은 치트 탐지도 쉽지 않았다.  
그리고 호스트는 다른 플레이어에 비해 대기 시간이 낮기 때문에 종종 우위에 있다. 전용 서버 모델을 채용함으로써 게임의 중단이나 치트의 문제를 줄이고, 모든 플레이어에게 평등한 게임 플레이 경험을 제공 할 수 있다.  
이것이 IllFonic가 Amazon GameLift에서 전용 서버를 가동 시킨다는 선택되어 멀티 플랫폼 (PC, Xbox One, PS4) 간의 플레이를 실현하기에 이르런 이유이다.  
  
    
## Ubisoft와 IllFonic에 의해 P2P와 전용 서버의 지식이 GDC 2018에서 공유 되었다
GameLift 팀에 의한 [Exploring Trends of Multiplayer Game Infrastructure with Amazon Gamelift](https://www.gdcvault.com/browse/gdc-18/play/1024829) 세션을 꼭 확인하기 바란다.  
  
Ubisoft의 For Honor 개발자 Damien Kieken와 Roman Campos Oriola이 IllFonic의 Friday the 13th The Game의 기술 디렉터 Paul Jackson에 의한 그들의의 백엔드 인프라나 P2P 네트워크에서 Amazon GameLift의 전용 게임 서버로의 이행에 대하여 소개하였다. 또한 Amazon GameLift 팀에서도 멀티 플레이어 게임 서버의 동향과 기계 학습 등도 소개하고 있다.  
  
GDC 2018 @ Amazon 관해서는 여기를 참조하자. aws.amazon.com/gdc-2018/schedule   
Amazon GameLift 대한 자세한 내용은 여기를 참조하자. aws.amazon.com/jp/gamelift   
  