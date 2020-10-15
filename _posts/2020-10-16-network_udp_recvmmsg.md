---
layout: post
title: UDP에서 recvmmsg를 사용하여 고성능 Receive 처리하기
published: true
categories: [Network]
tags: recvmmsg udp socket linux network
---
recvmmsg 라는 함수를  
http://www.jp.square-enix.com/conference/2018/onlinetech/pdf/20180421_ishimori.pdf  
라는 문서에서 알게 되었다.  
  
Linux에는 **recvmmsg** 라는 함수를 사용할 수 있고, 시스템 호출 1회로 대량의 패킷을 얻어올 수가 있다.  
1024개 패킷을 한꺼번에 얻는다면 시스템 호출로 인한 부하가 그만큼 줄게된다.  
  
마찬가지로 **sendmmsg** 라는 함수도 있다. 적절히 이용하자.  
  
1만명 접속X10pps 패킷을 recvfrom으로 하나씩 처리하다 보면, 패킷처리 만으로 CPU 파워의 50% 이상을 사용해 버릴 수도 있다.  
  
  
  
## recvmmsg에 대해
[출처](https://blog.tjun.org/entry/2014/02/08/recvmmsg%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6 )  
  
recvmmsg 라는 것은 recvmsg(2)를 확장하여 여러 메시지를 socket에서 받을 수 있는 시스템 호출로 Linux의 2.6.33에서 사용할 수 있다. glibc의 version 2.12에서 지원한다.   
  
이것을 사용하면, 즉 1 번 시스템 호출로 여러 번 recvmsg을 할 수 있도록 message를 왕창 받을 수 있으므로 응용 프로그램에서 성능이 올라갈 수 있다.  
  
man에있는 샘플 코드를 하단에 붙였다. 복수 개를 받을 뿐만 아니라 timeout을 취할 수 있는 것이 recvmsg과 다른 것으로 man을 읽으면 보통의 타임 아웃처럼 보이지만, 커널 소스를 읽으면 엄밀한 타임 아웃은 아닌 것 같다(라는 것을 가르쳐 주었다)  
  
또한 신속하게 빠져 나오기를 바랄 때 timeout에 0(tvsec = 0, tvnsec = 0)를 설정 하면, 하나 recv 한 것만으로 반환 되므로 주의. 즉시 빠져 나오고 싶어하지만, 이 때 가져올 message는 모두 가지 오고 싶을 때는 flag에 MSG_DONTWAIT를 설정하고 timeout은 NULL로 호출하면 원하는 대로 동작한다.  
  
또한 send에서도 비슷한 일을 하는 sendmmsg 라는 것도 있다.  
  
다음은 man에 있던 샘플 코드이다.  
```
#define _GNU_SOURCE
#include <netinet/ip.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>

int  
main(void)  
{
#define VLEN 10
#define BUFSIZE 200
#define TIMEOUT 1
  int sockfd, retval, i;
  struct sockaddr_in sa;
  struct mmsghdr msgs[VLEN];
  struct iovec iovecs[VLEN];
  char bufs[VLEN][BUFSIZE+1];
  struct timespec timeout;

  sockfd = socket(AF_INET, SOCK_DGRAM, 0);
  if (sockfd == -1) {
    perror("socket()");
    exit(EXIT_FAILURE);
  }

  sa.sin_family = AF_INET;
  sa.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
  sa.sin_port = htons(1234);
  if (bind(sockfd, (struct sockaddr *) &sa, sizeof(sa)) == -1) {
    perror("bind()");
    exit(EXIT_FAILURE);
  }

  memset(msgs, 0, sizeof(msgs));
  for (i = 0; i < VLEN; i++) {
    iovecs[i].iov_base = bufs[i];
    iovecs[i].iov_len = BUFSIZE;
    msgs[i].msg_hdr.msg_iov = &iovecs[i];
    msgs[i].msg_hdr.msg_iovlen = 1;
  }

  timeout.tv_sec = TIMEOUT;
  timeout.tv_nsec = 0;

  retval = recvmmsg(sockfd, msgs, VLEN, 0, &timeout);
  if (retval == -1) {
    perror("recvmmsg()");
    exit(EXIT_FAILURE);
  }

  printf("%d messages received\n", retval);
  for (i = 0; i < retval; i++) {
    bufs[i][msgs[i].msg_len] = 0;
    printf("%d %s", i+1, bufs[i]);
  }
  exit(EXIT_SUCCESS);
}


man 페이지에 있는 sendmmsg의 샘플 코드
#define _GNU_SOURCE
#include <netinet/ip.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>

int
main(void)
{
   int sockfd;
   struct sockaddr_in sa;
   struct mmsghdr msg[2];
   struct iovec msg1[2], msg2;
   int retval;

   sockfd = socket(AF_INET, SOCK_DGRAM, 0);
   if (sockfd == -1) {
	   perror("socket()");
	   exit(EXIT_FAILURE);
   }

   sa.sin_family = AF_INET;
   sa.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
   sa.sin_port = htons(1234);
   if (connect(sockfd, (struct sockaddr *) &sa, sizeof(sa)) == -1) {
	   perror("connect()");
	   exit(EXIT_FAILURE);
   }

   memset(msg1, 0, sizeof(msg1));
   msg1[0].iov_base = "one";
   msg1[0].iov_len = 3;
   msg1[1].iov_base = "two";
   msg1[1].iov_len = 3;

   memset(&msg2, 0, sizeof(msg2));
   msg2.iov_base = "three";
   msg2.iov_len = 5;

   memset(msg, 0, sizeof(msg));
   msg[0].msg_hdr.msg_iov = msg1;
   msg[0].msg_hdr.msg_iovlen = 2;

   msg[1].msg_hdr.msg_iov = &msg2;
   msg[1].msg_hdr.msg_iovlen = 1;

   retval = sendmmsg(sockfd, msg, 2, 0);
   if (retval == -1)
	   perror("sendmmsg()");
   else
	   printf("%d messages sent\n", retval);

   exit(0);
}
```  
  


