---
layout: post
title: Windows 10 에서 Unix Domain Socket 사용 가능
published: true
categories: [Network]
tags: AF_UNIX c++ socket windows10
---
Windows 10 2018년 봄 대형 업데이트에서 Unix Domain Socket 기능이 추가 되었다.  
[AF_UNIX comes to Windows – Windows Command Line Tools For Developers](https://blogs.msdn.microsoft.com/commandline/2017/12/19/af_unix-comes-to-windows/)  
Unix Domain Socket을 사용하기 위해서는 AF_UNIX를 사용하면 된다.    
  
아래 예제 코드의 [출처](https://mattn.kaoriya.net/software/lang/c/20180513001637.htm)    
코드를 보면 알겠지만 조금의 #ifdef로 windows, linux 지원 네트워크 프로그램을 만들 수 있다.   
서버  
```
#ifdef _WIN32
# include <ws2tcpip.h>
# include <io.h>
#else
# include <sys/fcntl.h>
# include <sys/types.h>
# include <sys/socket.h>
# include <netinet/in.h>
# include <netdb.h>
# define closesocket(fd) close(fd)
#endif
#include <stdio.h>

#define UNIX_PATH_MAX 108

typedef struct sockaddr_un {
  ADDRESS_FAMILY sun_family;
  char sun_path[UNIX_PATH_MAX];
} SOCKADDR_UN, *PSOCKADDR_UN;

int
main(int argc, char* argv[]) {
  int server_fd;
  int client_fd;
  struct sockaddr_un server_addr; 
  size_t addr_len;

#ifdef _WIN32
  WSADATA wsa;
  WSAStartup(MAKEWORD(2, 2), &wsa);
#endif

  unlink("./server.sock");

  if ((server_fd = socket(AF_UNIX, SOCK_STREAM, 0)) < 0) {
    perror("server: socket");
    exit(1);
  }

  memset((char *) &server_addr, 0, sizeof(server_addr));
  server_addr.sun_family = AF_UNIX;
  strcpy(server_addr.sun_path, "./server.sock");

  if (bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
    perror("server: bind");
    exit(1);
  }

  if (listen(server_fd, 5) < 0) {
    perror("server: listen");
    closesocket(server_fd);
    exit(1);
  }

  while (1) {
    if ((client_fd = accept(server_fd, NULL, NULL)) < 0) {
      perror("server: accept");
      break;
    }
    while (1) {
      char buf[256];
      int n = recv(client_fd, buf, sizeof(buf), 0);
      if (n <= 0) break;
      buf[n] = 0;
      puts(buf);
    }
    closesocket(client_fd);
  }

  closesocket(server_fd);

#ifdef _WIN32
  WSACleanup();
#endif
}
```
  
클라이언트  
```
#ifdef _WIN32
# include <ws2tcpip.h>
# include <io.h>
#else
# include <sys/fcntl.h>
# include <sys/types.h>
# include <sys/socket.h>
# include <netinet/in.h>
# include <netdb.h>
# define closesocket(fd) close(fd)
#endif
#include <stdio.h>

#define UNIX_PATH_MAX 108

typedef struct sockaddr_un {
  ADDRESS_FAMILY sun_family;
  char sun_path[UNIX_PATH_MAX];
} SOCKADDR_UN, *PSOCKADDR_UN;

int
main(int argc, char* argv[]) {
  int client_fd;
  struct sockaddr_un client_addr; 

#ifdef _WIN32
  WSADATA wsa;
  WSAStartup(MAKEWORD(2, 2), &wsa);
#endif

  if ((client_fd = socket(AF_UNIX, SOCK_STREAM, 0)) < 0) {
    perror("server: socket");
    exit(1);
  }

  memset((char *) &client_addr, 0, sizeof(client_addr));
  client_addr.sun_family = AF_UNIX;
  strcpy(client_addr.sun_path, "./server.sock");

  if (connect(client_fd, (struct sockaddr *)&client_addr, sizeof(client_addr)) < 0) {
    perror("client: connect");
    exit(1);
  }

  while (1) {
    char buf[256];
    if (!fgets(buf, sizeof(buf), stdin))
      break;
    char *p = strpbrk(buf, "\r\n");
    if (p) *p = 0;
    if (send(client_fd, buf, strlen(buf), 0) < 0)
      break;
  }

  closesocket(client_fd);

#ifdef _WIN32
  WSACleanup();
#endif
}
```  
    
MS가 점점 linux 프로그램이 윈도우에 쉽게 포팅 될 수 있는 기능을 만들고 있다.      