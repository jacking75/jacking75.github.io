---
layout: post
title: socket 프로그래밍을 할 때의 SIGPIPE에 의한 문제
published: true
categories: [Network]
tags: c++ socket linux sigpipe
---
[출처](http://doi-t.hatenablog.com/entry/2014/06/10/033309 )  
  
연결이 끊어진 소켓에 쓰기(send(2) 혹은 write(2) 같은 것)를 하면 SIGPIPE 시그널이 발생하고 프로세스가 종료하기 때문에 제대로 SIGPIPE 시그널을 핸들링 해 두자는 이야기 이다.  
(Windows OS에서는 상관 없는 이야기이다)  
  
## SISGPIPE를 받은 프로세스의 행동과 소켓 프로그래밍에서의 대응책
"sigpipe"로 검색하면 같은 이야기는 얼마든지 글이 있다. 예를 들면,  
> 내가 쓴 서버가 갑자기 사망하는 거예요 이유는 무엇입니까?' 라는 질문을 받을 수 있다. 이것은 종종 SIGPIPE 처리를 잊은 것이 원인이다. SIGPIPE는 절단된 네트워크 소켓 등에 데이터를 쓰려고 했을 때 전달되는 UNIX 시그널이다. 특히 설정하지 않으면, 프로세스는 SIGPIPE를 받으면 종료한다. 따라서 통신이 갑자기 끊어 질 수있는 TCP 서버에서는 SIGPIPE를 무시하도록 설정해야 한다.
  
출처: http://gihyo.jp/dev/serial/01/perl-hackers-hub/000603   
  
라고 설명되어 있기도하고, 비교적 자주 듣는 이야기 같다.  
  
SIGPIPE 시그널을 받은 프로세스의 기본 동작은 core 파일 조차 뱉지 않고 프로세스 종료 이다.  
이 동작은 위 인용대로 소켓 프로그래밍을 하고 있는 측면에서는 아무런 오류 출력도 없이 갑자기 죽어버리는 것이다.  
  
대응으로는(이것도 위 인용대로), 예를 들어 이전의 Signal 함수를 사용한다면 `Signal(SIGPIPE, SIG_IGN);` 라는 것을 프로그램 시작 직후에 넣어두는 것만으로 좋다.  
  
이제 send 실행 시 연결이 끊어져 있어도 프로세스를 종료하지 않고 단순히 send(2)가 -1을 반환한다.  
  
  
## 프로세스의 종료 상태와 SIGPIPE의 발생을 확인
여기에서는 다음의 샘플로 실제 파이프의 기입 처를 kill 하여  SIGPIPE의 발생을 유사 체험 해 본다.  
소켓 프로그래밍적으로는 아래의 sender.sh가 송신자 (write (2), send (2)), receiver.sh가 수신자 (read (2), recv (2))라고 생각해 주면 좋다.  
즉 수신이 중단된 곳에 송신자가 wirte(2)를 실행 했을 때 어떻게 되는지를 본다.  
![](/images/2019/select_sigpipe01.png)  
  
strace 명령으로 SIGPIPE의 발생을 확인  
SIGPIPE의 발생 확인은 strace 명령의 결과와 프로세스의 종료 상태 에서 하려고 생각한다.  
참고: [strace 명령의 사용법을 정리해 보았다](http://blog.livedoor.jp/sonots/archives/18193659.html ) 
  
다음은 위 예제 중 sigpipe.bash을 실행하여 이 때 출력되는 trace.log의 끝(이 장의 논의에 필요한 부분만)를 출력한 결과이다.  
![](/images/2019/select_sigpipe02.png)  
  
먼저 strace 명령의 결과를 보면, write(2)가 실패하고 있다( EPIPE (Broken pipe))을 확인할 수있다. 프로세스는 killed by SIGPIPE.  
종료 상태의 결과를 출력하고 `${PIPESTATUS[@]}` 는 bash 고유의 기능이다 (편리하다!).  
  
`exit status:141 143` 는 sender.sh이 SIGPIPE에 의해 종료(141) 한 것을, receiver.sh이 SIGTERM 종료(143) 한 것을 의미하고 있다.  
적어도 Linux 는 128에 시그널 번호를 더한 것을 종료 상태로 하고 있는 것 같다.  
  
Posix 에서는 반환 값은 128 이상이어야 한다고 정해져 있을뿐이지만 , Linux 는 128+ 시그널 번호를 반환한다.  
Linux의 표준 신호의 값을 확인하면 SIGPIPE : 13, SIGTERM : 15 이다.  
  
reciever.sh는 kill 명령에서 SIGTERM 전송하고 종료하고있다. 즉 128 + 15 = 143  
sender.sh는 대상 프로세스가 죽기 때문에 SIGPIPE 시그널이 발생하고 종료한다. 즉 128 + 13 = 141  
종료 상태의 의미적으로도 확실히 SIGPIPE로 다운하고 있는지 확인할 수 있다.  
  
아무튼 가장 큰 교훈은 다른 방식의 디버깅 방법을 항상 복수 소지 해두자 라는 것 같은 생각이 든다.  
  
  
**추가(최흥배)  **
C++ 오픈 소스 라이브러리 Poco를 보면 위의 문제는 Unix 계열에서만 발생하는 문제이고, Linux에서는 발생하지 않는다고 한다.
![](/images/2019/select_sigpipe03.png)  
  