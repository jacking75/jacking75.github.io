---
layout: post
title: C 언어로 Simple HTTP 서버 만들기
published: true
categories: [Network]
tags: network server http c socket
---
[출처](https://reerishun.com/makerblog/?p=712 )  
기능은 아주 단순하다. 그만큼 분석하기 쉽다. 웹서버의 대략적인 동작을 엿볼 수 있다.  
  
간단하게 아래의 기능만 구현했다.
- HTTP 서버
- HTML 출력
- 라우팅
  
  
## 환경
아래 환경에서 개발, 테스트 했다.
- OS : Windows 10
- IDE : Visual Studio 2015
  
  
  
## HTML을 표시하기
가장 유명한 HelloWorld를 해 본다.  
HTML 파일을 만들고, 1행씩 보내도 좋지만 이번은 귀찮아서 간단하게 직접 문장을 보낸다.  
```
strcpy(html,
      "<!DOCTYPE html>\n"
      "<html lang = \"ja\">\n"
      "<head>\n"
      "<meta charset = \"utf-8\">\n"
      "</head>\n"
      "<body>\n"
      "<h1>HelloWorld</h1>\n"
      "</body>"
      "</html>");
 
// 응답（HTML을 보낸다）
if (send(sockw, html, strlen(html), 0) < 1)
{
  printf("send : %d\n", WSAGetLastError());
  break;
}
```
  
미리 만들어 놓은 html 배열에 html를 저장하고, 그대로 보내면 브라우저에 HTML이 출력된다.  
  
  
  
## 라우팅 해 본다.
요청에 따라 다른 HTML을 보내도록 라우팅을 만들어본다.  
라우팅은 특정 주소에 액세스 할 때 이 목적 주소 결과를 출력한다.   
  
실제로 작성한 코드는 이런 식이다.  
```
//접속
recv_len = recvfrom(sockw, buf,1024, 0, (struct sockaddr *)&client, &sockaddr_in_size);
buf[recv_len - 1] = 0;
if (buf[0] == '\0')
  strcpy(buf, NULL);


// 통신 표시
printf("%s \n",buf);

// method
for (int i = 0; i < strlen(buf); i++) {
  printf("%d\n",i);
  if (buf[i] == 'G' && buf[i + 1] == 'E' && buf[i + 2] == 'T' && buf[i + 3] == ' ') {
  for (int j = 0; buf[i + 4 + j] != ' '; j++) {
    path[j] = buf[i + 4 + j];
  }
  break;
  }else if (buf[i] == 'P' && buf[i + 1] == 'O' && buf[i + 2] == 'S' && buf[i + 3] == 'T' && buf[i + 4] == ' ') {
  for (int j = 0; buf[i + 4 + j] != ' '; j++) {
    path[j] = buf[i + 4 + j];
  }
  break;
  }
}
```
  
이 코드에서 GET/POST의 값을 path 변수에 저장한다.  
path 변수라고 말하면 왠지 헷가릴 수 있으므로 이름은 다시 한번 바꾸어 둡다.  
```
// 라우팅
if (strcmp(path, "/page1") == 0) {
  strcpy(html,
  "<!DOCTYPE html>\n"
  "<html lang = \"ja\">\n"
  "<head>\n"
  "<meta charset = \"utf-8\">\n"
  "</head>\n"
  "<body>\n"
  "<h1>Page1</h1>\n"
  "<a href=\"/page2\">->page2</a>\n"
  "</body>"
  "</html>");
}else if (strcmp(path, "/page2") == 0) {
  strcpy(html,
  "<!DOCTYPE html>\n"
  "<html lang = \"ja\">\n"
  "<head>\n"
  "<meta charset = \"utf-8\">\n"
  "</head>\n"
  "<body>\n"
  "<h1>Page2</h1>\n"
  "<a href=\"/page1\">->page1</a>\n"
  "</body>"
  "</html>");
}else {
  strcpy(html,
  "<!DOCTYPE html>\n"
  "<html lang = \"ja\">\n"
  "<head>\n"
  "<meta charset = \"utf-8\">\n"
  "</head>\n"
  "<body>\n"
  "<h1>Not Found</h1>\n"
  "</body>"
  "</html>");
}

// 응답（HTML을 보낸다）
if (send(sockw, html, strlen(html), 0) < 1)
{
  printf("send : %d\n", WSAGetLastError());
  break;
}
```
    
이렇게하면 http://localhost:8000/page1 에 접속하면 페이지가 표시된다.  
앵커로 http://localhost:8000/page2 로 이동할 수 있다.  
  
또 다른 페이지를 열려고하면 "Not Found" 라고한다.  
  
  
  
전체 코드는 아래와 같다.  
main.cpp    
```
#include <windows.h>
#include <stdio.h>
#include <conio.h>
#include <tchar.h>
 
WSADATA     wsaData;
SOCKET      sock0;
SOCKET      sockw;
struct      sockaddr_in addr;
struct      sockaddr_in client;
#define PORT_NUM 8000
 
 
// IP 주소 얻기
int getAddrHost(void) {
  int i;
  HOSTENT *lpHost;       //  호스트 정보를 저장하는 구조체
  IN_ADDR inaddr;       // IP 주소를 저장하는 구조체
  char szBuf[256], szIP[16];  // 호스트 이름/IP 주소를 저장하는 배열
 
  // 에러 처리
  if (WSAStartup(MAKEWORD(1, 1), &wsaData) != 0) {
    printf("WSAStartup Error\n");
    return -1;
  }
 
  // 로컬 머신의 호스트 이름을 얻는다
  gethostname(szBuf, (int)sizeof(szBuf));
 
  // 호스트 정보 얻기
  lpHost = gethostbyname(szBuf);
  // IP 주소 얻기
  for (i = 0; lpHost->h_addr_list[i]; i++) {
    memcpy(&inaddr, lpHost->h_addr_list[i], 4);
  }
  strcpy(szIP, inet_ntoa(inaddr));
  printf("build server: http://%s:%d\n", szIP,PORT_NUM);  
 
  WSACleanup();
 
  return 0;
}
 
char *get_name(char *name) {
  
}
 
 
int main()
{
  int len;
  int n;
  int sockaddr_in_size = sizeof(struct sockaddr_in);
  int recv_len = 0;
  unsigned char buf[1024];
  unsigned char path[1024];
  unsigned char html[1024];
 
  //IP 어드레스 표시
  if (getAddrHost() != 0) {
    printf("get IP address failed");
    getch();
    return -1;
  }
 
  // winsock2 초기화
  if (WSAStartup(MAKEWORD(2, 0), &wsaData))
  {
    puts("reset winsock failed");
    getch();
    return -1;
  }
 
  // 소켓 만들기
  sock0 = socket(AF_INET, SOCK_STREAM, 0);
  if (sock0 == INVALID_SOCKET)
  {
    printf("socket : %d\n", WSAGetLastError());
    getch();
    return -1;
  }
 
  // 소켓 설정
  addr.sin_family = AF_INET;
  addr.sin_port = htons(PORT_NUM);
  addr.sin_addr.S_un.S_addr = INADDR_ANY;
 
  // 소켓 바인드
  if (bind(sock0, (struct sockaddr *)&addr, sizeof(addr)) != 0)
  {
    printf("bind : %d\n", WSAGetLastError());
    getch();
    return -1;
  }
 
  // TCP 클라이언트로부터의 접속 요구를 기다리면서 대기한다
  if (listen(sock0, 5) != 0)
  {
    printf("listen : %d\n", WSAGetLastError());
    getch();
    return -1;
  }
 
 
  // 서버 시작
  while (1)
  {
 
    len = sizeof(client);
    sockw = accept(sock0, (struct sockaddr *)&client, &len);
    if (sockw == INVALID_SOCKET)
    {
      printf("accept : %d\n", WSAGetLastError());
      break;
    }
 
    // 버퍼 초기화
      memset(path, 0, 1024);
    memset(html, 0, 1024);
 
    // 접속 
    recv_len = recvfrom(sockw, buf,1024, 0, (struct sockaddr *)&client, &sockaddr_in_size);
    buf[recv_len - 1] = 0;
    if (buf[0] == '\0')
      strcpy(buf, NULL);
 
 
    // 통신 표시
    printf("%s \n",buf);
 
    // method
    for (int i = 0; i < strlen(buf); i++) {
      printf("%d\n",i);
      if (buf[i] == 'G' && buf[i + 1] == 'E' && buf[i + 2] == 'T' && buf[i + 3] == ' ') {
        for (int j = 0; buf[i + 4 + j] != ' '; j++) {
          path[j] = buf[i + 4 + j];
        }
        break;
      }else if (buf[i] == 'P' && buf[i + 1] == 'O' && buf[i + 2] == 'S' && buf[i + 3] == 'T' && buf[i + 4] == ' ') {
        for (int j = 0; buf[i + 4 + j] != ' '; j++) {
          path[j] = buf[i + 4 + j];
        }
        break;
      }
    }
    printf("request: %s \n",path);
 
    // HTTP
    unsigned char *header =  
      "HTTP/1.0 200 OK\n"
      "Content-type: text/html\n"
      "\n";
 
    send(sockw, header, strlen(header), 0);
 
    // 라우팅 
    if (strcmp(path, "/page1") == 0) {
      strcpy(html,
        "<!DOCTYPE html>\n"
        "<html lang = \"ja\">\n"
        "<head>\n"
        "<meta charset = \"utf-8\">\n"
        "</head>\n"
        "<body>\n"
        "<h1>Page1</h1>\n"
        "<a href=\"/page2\">->page2</a>\n"
        "</body>"
        "</html>");
    }else if (strcmp(path, "/page2") == 0) {
      strcpy(html,
        "<!DOCTYPE html>\n"
        "<html lang = \"ja\">\n"
        "<head>\n"
        "<meta charset = \"utf-8\">\n"
        "</head>\n"
        "<body>\n"
        "<h1>Page2</h1>\n"
        "<a href=\"/page1\">->page1</a>\n"
        "</body>"
        "</html>");
    }else {
      strcpy(html,
        "<!DOCTYPE html>\n"
        "<html lang = \"ja\">\n"
        "<head>\n"
        "<meta charset = \"utf-8\">\n"
        "</head>\n"
        "<body>\n"
        "<h1>404- Not Found</h1>\n"
        "</body>"
        "</html>");
    }
 
    // 응답（HTML을 보낸다）
    if (send(sockw, html, strlen(html), 0) < 1)
    {
      printf("send : %d\n", WSAGetLastError());
      break;
    }
 
    // 소켓 닫기
    closesocket(sockw);
  }
 
  // winsock2 종료 처리
  closesocket(sock0);
  WSACleanup();
  return 0;
}
```
  