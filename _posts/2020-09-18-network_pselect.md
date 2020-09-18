---
layout: post
title: select() 보다는 pselect()를 사용하는 것이 좋다
published: true
categories: [Network]
tags: pselect c c++ socket
---
[pselect ?](https://blog.naver.com/leedoosol/220895159001)  
[다중 입출력 함수 - pselect()](https://neakoo35.blog.me/30132362715)  
[(일어)pselect로 시그널을 무시한다](http://www.geekpage.jp/programming/linux-network/pselect-eintr.php)  
  
  
## 예제 코드 - pselect
  
```  
#include <stdio.h>
#include <signal.h>
#include <sys/select.h>
 
void
sigusr1_handler(int sig)
{
  printf("signal USR1 called\n");
}
 
void
sigusr2_handler(int sig)
{
  printf("signal USR2 called\n");
}
 
int
main()
{
  fd_set readfds;
  int n;
  sigset_t sigset;
 
  /* SIGUSR1의 시그널 핸들러를 설정 */
  signal(SIGUSR1, sigusr1_handler);
 
  /* SIGUSR2의 시그널 핸들러를 설정 */
  signal(SIGUSR2, sigusr2_handler);
 
  /* SIGUSR1의 시그널 설정 */
  sigemptyset(&sigset);
  sigaddset(&sigset, SIGUSR1);
 
  FD_ZERO(&readfds);
 
  printf("before select\n");
 
  /* SIGUSR2가 올 때까지 select는 계속 block 한다 */
  /* SIGUSR1 에서는 EINTR을 반환하지 않는다 */
  n = pselect(0, &readfds, NULL, NULL, NULL, &sigset);
  printf("after select : %d\n", n);
 
  /* errno가 EINTR로 되어 있는지를 확인하자 */
  perror("after select");
 
  return 0;
}
```
  
<pre>
kill -s SIGUSR1 애플리케이션PID
</pre>
  
경우는 해당 시그널 무시  
  
kill -s SIGUSR2 애플리케이션PID  
<pre>
before select
signal USR2 called
signal USR1 called
after select : -1
after select: Interrupted system call
</pre>
  
  
## 예제 코드 - select
  
```
#include <stdio.h>
#include <signal.h>
#include <sys/select.h>
 
void
sigusr1_handler(int sig)
{
  printf("signal called\n");
}
 
int
main()
{
  fd_set readfds;
  int n;
 
  /* SIGUSR1의 시그널 핸들러를 설정*/
  signal(SIGUSR1, sigusr1_handler);
 
  FD_ZERO(&readfds);
 
  printf("before select\n");
 
  /* SIGUSR1이 올 때까지 select는 계속 block 한다 */
  n = select(0, &readfds, NULL, NULL, NULL);
  printf("after select : %d\n", n);
 
  // errno가 EINTR이 된 것을 확인해보자. 이것은 에러가 아니다
  perror("after select");
 
  return 0;
}
```
  
<pre>
kill -s SIGUSR1 애플리케이션PID
</pre>  
  
결과  
<pre>
before select
signal called
after select : -1
after select: Interrupted system call
</pre>
  
만약 위의 시그널를 핸들링 하지 않으면 바로 종료한다  
<pre>
before select
사용자 정의 시그널 1
</pre>
  


