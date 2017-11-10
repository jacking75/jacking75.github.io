---
layout: post
title: Win32 API - 대기 체인 분석 
published: true
categories: [Network]
tags: network virtualbox rdp
--- 
Windows7 에서 새로 생긴 기능이다.  
이것을 사용하면 프로그램을 종료 시켰는데는 프로세스는 죽지 않고 살아 있는 경우 어디서 문제가 되었는지 어느 정도 알아낼 수 있다.  
  
![](/images/2017/0928-001.PNG)   
이 프로그램은 내가 집에서 만든 서버 애플리케이션으로 X 버튼을 눌러서 종료시킨다.  
  
![](/images/2017/0928-002.PNG)   
그런데 작업 관리자를 보면 프로세스가 아직 죽지 않았다(ProjectHF2.exe)  
  
이 문제는 보통 워커 스레드를 사용하는 경우 특정 스레드가 죽지 않아서 발생하는 경우이다.  
그럼 어떤 스레드가 대기 상태에 있는지 알기 위해서 대기 체인 분석을 사용해 보겠다.  
  
![](/images/2017/0928-003.PNG)   
리소스 버튼을 누르면 아래의 창이 나온다.  
  
![](/images/2017/0928-004.PNG)   
위 그림과 같이 프로세스를 선택 후 메뉴에서 '대기 체인 분석'을 선택한다. 
  
![](/images/2017/0928-005.PNG)   
위와 같은 화면이 나오고 어떤 스레드가 어떤 이유로 대기하고 있는지 설명해준다.  
  
이 기능은 Windows API 중 Wait Chain Traversal를 사용한 것이다.  
http://msdn.microsoft.com/ko-KR/library/ms681622(VS.85).aspx  
  
어느 정도 제한은 있지만 이 API를 사용하면 데드락 상황도 판단할 수 있다.  
  
    