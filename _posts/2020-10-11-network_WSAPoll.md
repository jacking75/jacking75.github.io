---
layout: post
title: WSAPoll
published: true
categories: [Network]
tags: network server WSAPoll c socket Windows
---
## 개요
- Windows Vista부터 새로 생긴 네트워크 API
- UNIX(Linux) OS의 poll과 비슷한 것이다
- 복수의 파일 디스크립터를 감시하는 API
- 지정한 소켓의 상태가 변화했는지 확인하는 기능을 제공한다. 기능적으로는 select와 유사하다
  
  
## API 설명
[WSAPoll function](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741669.aspx)  
  
```
int WSAAPI WSAPoll(
  WSAPOLLFD fdarray[],
  ULONG nfds,
  INT timeout
);
```
  
fdarray는 WSAPOLLFD 구조체 배열을 지정한다.  
nfds는 fdarray의 요소 수를 지정한다.  
timeout은 함수 호출 후 대기할 시간이다(어떤 이벤트도 없으면 이 시간까지 대기한다).  
0 보다 클 경우 그 값만큼 함수가 대기하고 0의 경우는 즉시 제어를 반환한다.  
마이너스 값의 경우는 소켓이 지정된 위상에 변화가 있을 때까지 제어를 반환하지 않는다.  
  
```
typedef struct pollfd {
  SOCKET fd;
  short  events;
  short  revents;
} WSAPOLLFD
```
  
events는 어떤 이벤트 발생을 원하는지를 뜻하는 플래그 셋이다. 이것은 꼭 하나 이상 설정이 되어야 한다.  
  
| Flag    | Description |
|------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| POLLPRI    | Priority data를 블러킹 없이 읽을 수 있다. 이 플로그는 MS winsock에서는 지원하지 않는다. |
| POLLRDBAND | Priority band(out-of-band) data를 블럭킹 읽을 수 있다.                                                                |
| POLLRDNORM | 보통의 데이터를 블럭킹 없이 읽을 수 있다.                                                                              |
| POLLWRNORM | 보통의 데이터를 블럭킹 없이 쓸 수 있다.                                                                                    |
  
POLLIN 플러그는 POLLRDNORM과 POLLRDBAND 플러그의 값을 합친 것이다.  
POLLOUT 플러그는 POLLWRNORM 플러그 값과 같은 정의이다.  
  
<br>  
  
revents는 WSAPoll 함수 호출 후 반환 되었을 때 상태 쿼리 결과를 나타내는 플러그 셋이다. 이것은 아래의 플러그를 조합할 수 있다.  
  
| Flag    | Description                                                                                                     |
|------------|----------------------------------------------------------------------------------------------------------------------------|
| POLLERR    | 에러가 발생하였다.                                                                                                     |
| POLLHUP    | 스트림 지향 접속의 경우 접속 종료 및 중지 되었다                                                                         |
| POLLNVAL   | 무효한 소켓이 사용되었다.                                                                                      |
| POLLPRI    | Priority data를 블럭킹 없이 읽을 수 있다. 이 플로그는 MS winsock에서는 지원하지 않는다. |
| POLLRDBAND | Priority band(out-of-band) data를 블럭킹 없이 읽을 수 있다                       |
| POLLRDNORM | 보통의 데이터를 블럭킹 없이 읽을 수 있다.                                                    |
| POLLWRNORM | 보통의 데이터를 블럭킹 없이 쓸 수 있다                                                                       |
  
  
  
## 에코 TCP 서버
클라이언트 접속을 받은 후 받은 데이터를 그대로 보낸다.  
  
```
// 출처: http://eternalwindows.jp/network/winsock/winsock07s.html

#include <stdio.h>

#include <winsock2.h>
#include <ws2tcpip.h>

#pragma comment(lib, "ws2_32.lib")


HWND g_hwndListBox = NULL;
BOOL g_bExitThread = FALSE;

SOCKET InitializeWinsock(LPSTR lpszPort);
DWORD WINAPI ThreadProc(LPVOID lpParamater);
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);

int WINAPI WinMain(HINSTANCE hinst, HINSTANCE hinstPrev, LPSTR lpszCmdLine, int nCmdShow)
{
  TCHAR      szAppName[] = TEXT("sample-server");
  HWND       hwnd;
  MSG        msg;
  WNDCLASSEX wc;

  wc.cbSize = sizeof(WNDCLASSEX);
  wc.style = 0;
  wc.lpfnWndProc = WindowProc;
  wc.cbClsExtra = 0;
  wc.cbWndExtra = 0;
  wc.hInstance = hinst;
  wc.hIcon = (HICON)LoadImage(NULL, IDI_APPLICATION, IMAGE_ICON, 0, 0, LR_SHARED);
  wc.hCursor = (HCURSOR)LoadImage(NULL, IDC_ARROW, IMAGE_CURSOR, 0, 0, LR_SHARED);
  wc.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);
  wc.lpszMenuName = NULL;
  wc.lpszClassName = szAppName;
  wc.hIconSm = (HICON)LoadImage(NULL, IDI_APPLICATION, IMAGE_ICON, 0, 0, LR_SHARED);

  if (RegisterClassEx(&wc) == 0) {
    return 0;
  }

  hwnd = CreateWindowEx(0, szAppName, szAppName, WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, NULL, NULL, hinst, NULL);
  if (hwnd == NULL) {
    return 0;
  }

  ShowWindow(hwnd, nCmdShow);
  UpdateWindow(hwnd);

  while (GetMessage(&msg, NULL, 0, 0) > 0) {
    TranslateMessage(&msg);
    DispatchMessage(&msg);
  }

  return (int)msg.wParam;
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
  static HANDLE hThread = NULL;
  static SOCKET socListen = INVALID_SOCKET;

  switch (uMsg) {

  case WM_CREATE: {
    DWORD dwThreadId;

    g_hwndListBox = CreateWindowEx(0, TEXT("LISTBOX"), NULL, WS_CHILD | WS_VISIBLE | WS_VSCROLL, 0, 0, 0, 0, hwnd, (HMENU)1, ((LPCREATESTRUCT)lParam)->hInstance, NULL);

    socListen = InitializeWinsock("32452");
    if (socListen == INVALID_SOCKET)
      return -1;

    hThread = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)ThreadProc, &socListen, 0, &dwThreadId);

    return 0;
  }

  case WM_SIZE:
    MoveWindow(g_hwndListBox, 0, 0, LOWORD(lParam), HIWORD(lParam), TRUE);
    return 0;

  case WM_DESTROY:
    if (hThread != NULL) {
      g_bExitThread = TRUE;
      WaitForSingleObject(hThread, 1000);
      CloseHandle(hThread);
    }

    if (socListen != INVALID_SOCKET) {
      closesocket(socListen);
      WSACleanup();
    }

    PostQuitMessage(0);
    return 0;

  default:
    break;

  }

  return DefWindowProc(hwnd, uMsg, wParam, lParam);
}

DWORD WINAPI ThreadProc(LPVOID lpParamater)
{
  SOCKET    socListen = *((SOCKET *)lpParamater);
  int       i;
  int       nResult;
  int       nMaxSocketCount = 11;
  WSAPOLLFD fdArray[11];

  for (i = 0; i < nMaxSocketCount; i++) {
    fdArray[i].fd = INVALID_SOCKET;
    fdArray[i].events = 0;
  }

  fdArray[0].fd = socListen;
  fdArray[0].events = POLLRDNORM;

  while (!g_bExitThread) {
    nResult = WSAPoll(fdArray, nMaxSocketCount, 500);
    if (nResult < 0) {
      SendMessage(g_hwndListBox, LB_ADDSTRING, 0, (LPARAM)TEXT("WSAPOLLFD 실행이 실패하였다"));
      break;
    }
    else if (nResult == 0) {
      continue;
    }

    for (i = 0; i < nMaxSocketCount; i++) {
      if (fdArray[i].revents & POLLRDNORM)
        break;
      else if (fdArray[i].revents & POLLHUP)
        break;      
    }

    if (fdArray[i].revents & POLLRDNORM) {
      if (fdArray[i].fd == socListen) {
        int              nAddrLen;
        char             szBuf[256];
        char             szHostName[256];
        SOCKADDR_STORAGE sockAddr;

        for (i = 0; i < nMaxSocketCount; i++) {
          if (fdArray[i].fd == INVALID_SOCKET)
            break;
        }

        if (i == nMaxSocketCount)
          continue;

        nAddrLen = sizeof(SOCKADDR_STORAGE);
        fdArray[i].fd = accept(socListen, (LPSOCKADDR)&sockAddr, &nAddrLen);
        fdArray[i].events = POLLRDNORM;

        getnameinfo((LPSOCKADDR)&sockAddr, nAddrLen, szHostName, sizeof(szHostName), NULL, 0, 0);
        wsprintfA(szBuf, "No%d(%s) 접속", i, szHostName);
        SendMessageA(g_hwndListBox, LB_ADDSTRING, 0, (LPARAM)szBuf);
      }
      else {
        int   nResult;
        wchar_t szBuf[256];
        char szData[256] = { 0, };

        nResult = recv(fdArray[i].fd, (char *)szData, sizeof(szData), 0);

        char szSendMessage[256] = { 0, };
        sprintf_s(szSendMessage, 256 - 1, "Re:%s", szData);
        int nMsgLen = strnlen_s(szSendMessage, 256 - 1);

        wsprintf(szBuf, L"No%d %S", i, szSendMessage);
        SendMessage(g_hwndListBox, LB_ADDSTRING, 0, (LPARAM)szBuf);

        nResult = send(fdArray[i].fd, (char *)szSendMessage, nMsgLen, 0);
      }
    }
    else if (fdArray[i].revents & POLLHUP) {
      wchar_t szBuf[256];

      wsprintf(szBuf, L"No%d 접속 종료", i);
      SendMessage(g_hwndListBox, LB_ADDSTRING, 0, (LPARAM)szBuf);

      shutdown(fdArray[i].fd, SD_BOTH);
      closesocket(fdArray[i].fd);
      fdArray[i].fd = INVALID_SOCKET;
      fdArray[i].events = 0;
    }
  }

  return 0;
}

SOCKET InitializeWinsock(LPSTR lpszPort)
{
  WSADATA    wsaData;
  ADDRINFO   addrHints;
  LPADDRINFO lpAddrList;
  SOCKET     socListen;

  WSAStartup(MAKEWORD(2, 2), &wsaData);

  ZeroMemory(&addrHints, sizeof(addrinfo));
  addrHints.ai_family = AF_INET;
  addrHints.ai_socktype = SOCK_STREAM;
  addrHints.ai_protocol = IPPROTO_TCP;
  addrHints.ai_flags = AI_PASSIVE;

  if (getaddrinfo(NULL, lpszPort, &addrHints, &lpAddrList) != 0) {
    MessageBox(NULL, TEXT("호스트 정보에서 어드레스 취득이 실패하였다"), NULL, MB_ICONWARNING);
    WSACleanup();
    return INVALID_SOCKET;
  }

  socListen = socket(lpAddrList->ai_family, lpAddrList->ai_socktype, lpAddrList->ai_protocol);

  if (bind(socListen, lpAddrList->ai_addr, (int)lpAddrList->ai_addrlen) == SOCKET_ERROR) {
    MessageBox(NULL, TEXT("로컬 어드레스와 소켓 연결에 실패하였다"), NULL, MB_ICONWARNING);
    closesocket(socListen);
    freeaddrinfo(lpAddrList);
    WSACleanup();
    return INVALID_SOCKET;
  }

  if (listen(socListen, 1) == SOCKET_ERROR) {
    closesocket(socListen);
    freeaddrinfo(lpAddrList);
    WSACleanup();
    return INVALID_SOCKET;
  }

  freeaddrinfo(lpAddrList);

  return socListen;
}
```
  
  
프로그램에서는 클라이언트의 접속과 클라이언트가 보낸 데이터를 받기 위해서 WSAPoll을 사용하고 있다.  
  
```
int       nMaxSocketCount = 11;
WSAPOLLFD fdArray[11];

for (i = 0; i < nMaxSocketCount; i++) {
  fdArray[i].fd     = INVALID_SOCKET;
  fdArray[i].events = 0;
}

fdArray[0].fd     = socListen;
fdArray[0].events = POLLRDNORM;
```
  
fdArray의 요소 수는 11로 되어 있는데 이것은 1+10으로 1개의 리슨 소켓과 10개의 서버 소켓의 상태의 변경을 검출하는 목적이다.  
즉 접속할 수 있는 클라이언트의 최대 크기는 10개이다.  
초기 상태에서는 서버 소켓은 만들어지 않았으므로(접속한 클라이언트가 없으니) NULL을 지정하고 리슨 소켓은 배열의 선두에 지정한다.  
events는 검출하고 싶은 이벤트를 지정하고 POLLRDNORM은 접속 또는 데이터 읽기 검출을 한다.  
  
```
nResult = WSAPoll(fdArray, nMaxSocketCount, 500);
if (nResult < 0) {
  SendMessage(g_hwndListBox, LB_ADDSTRING, 0, (LPARAM)TEXT("WSAPOLLFD 실행이 실패하였다"));
  break;
}
else if (nResult == 0)
  continue;
```
  
WSAPoll이 0 이하의 값을 반환한 경우에는 함수가 실패했음을 의미하므로 그 뜻을 표시한다.   
0이 반환된 경우에는 타임 아웃이 발생한 것을 의미하며 이 경우 후속 처리를 실행하지 않는다.  
이 WSAPoll의 호출에서는 500밀리 초 타임 아웃 값을 지정하고 있지만 본래라면 -1 등의 마이너스 값을 지정하여 이벤트가 발생할 때까지 대기하는 것이 이상적이다.  
그러나 그럴 경우 대기 중에 g_bExitThread의 값을 확인할 수 없어서 스레드가 종료해야 할 때를 특정할 수 없게 되므로 타임 아웃을 지정하여 일정 간격으로 제어를 반환하도록 구현하고 있다. 0 보다 큰 값이 반환된 경우 배열 내의 어떤 소켓에 상태의 변경이 생겼는지 다음 처리에서 확인한다.  
  
```
for (i = 0; i < nMaxSocketCount; i++) {
  if (fdArray[i].revents & POLLRDNORM)
    break;
  else if (fdArray[i].revents & POLLHUP)
    break;
}
```
  
소켓의 현재 상태는 revents 멤버에 저장되어 있다. 이 값이 POLLRDNORM으로 있다는 것은 접속 또는 읽기가 발생한 것을 의미하기 때문에 루프를 빠져 나온다.  
POLLHUP은 접속 종료를 의미하는 것으로 이것도 중요한 상태 변경이므로 루프를 빠져 나온다.  
  
리슨 소켓에 대해서 POLLRDNORM이 저장되어 있다면 접속을 뜻하므로 accept를 호출, 그렇지 않다면 서버 소켓에 클라이언트의 데이터를 받을 수 있으므로 recv를 호출하여 수신한다.  
  
  
## 참고
http://eternalwindows.jp/network/winsock/winsock07s.html
