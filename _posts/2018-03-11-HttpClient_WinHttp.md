---
layout: post
title: C++ http 클라이언트 요청하기 WinHttp
published: true
categories: [Network]
tags: cpp http winhttp
---
아직은 기본 C++ 라이브러리로 http 통신을 할 수 없다.  
(아마 C++ 20 에서는 가능하지 않을까 생각한다).  
  
C++로 웹서버에 http 통신을 하려면 외부 라이브러리를 사용하던가 혹은 OS에서 제공하는 API를 사용해야 한다.  
이 글에서는 Windows 플랫폼 한정으로 Win32 API를 사용하여 http 요청을 보내는 방법을 간단하게 설명한다.  
  
<br>  
  
## WinHttp
Windows Vista 에서 새로 생긴 Win32API.  
Windows XP SP3부터 사용할 수 있다.  
  
### API  
  
**Winhttp.h** 헤더 파일을 포함하고, **Winhttp.lib** 라이브러리를 링크해야 한다.  
  
HTTP 서비스 함수를 호출하기 전에 WinHttpOpen 함수를 호출한다.  
이 함수에 이후의 Web 요구에서 보내는 유저 에이전트의 이름, 프록시(사용하는 경우)에 관한 정보 및 Web 호출이 동기적 또는 비동기적인가? 라는 정보를 준다.  
  
**WinHttpOpen** 함수는 세션 핸들(HINTERNET)을 반환 받는다.  
**이 핸들은 모든 HTTP 서비스 핸들과 마찬가지로 처리가 끝나면 WinHttpCloseHandle 함수를 사용하여 닫아야 한다**.  
  
다음으로 **WinHttpConnect** 함수를 호출하여 연결 핸들을 가져온다.  
이 함수에서 세션 핸들, 서버 이름 및 서버 포트를 지정한다.  
**이 함수는 네트워크 접속을 실행하는 것이 아니라 내부 접속 설정을 준비할 뿐이다.**  
  
다음으로 요구를 작성하기 위해 **WinHttpOpenRequest** 함수를 호출하여 연결 핸들과 실행하는 요구에 관한 정보를 준다.  
WinHttpOpenRequest 함수는 요구의 일부로서 송신하는 RFC822, MIME 및 HTTP의 모든 헤더를 저장하는데 사용되는 요구 핸들을 반환한다.  
요구의 실제 헤더를 추가하려면 **WinHttpAddRequestHeaders** 함수를 호출하여 복귀/개행 페어로 구별되는 모든 헤더를 포함한 문자열을 건네준다.    
  
실제 네트워크 호출은 **WinHttpSendRequest** 함수를 호출하여 요구 핸들을 전달한 때에 열린다.    
이 함수를 사용하여 WinHtttpAddRequestHeaders 호출에서 지정되지 않은 추가 HTTP 헤더 및 PUT 또는 POST 요구를 위한 호출에 필요한 추가 데이터를 지정할 수 있다.  
서버가 요구에 응답하면, 요구 핸들을 지정하여 **WinHttpReceiveResponse** 함수를 호출하는 시스템이 서버에서 응답 헤더를 들여다볼 수 있게 한다(응답 헤더는 WinHttpQueryHeaders 함수를 호출함으로써 얻을 수 있다).  
  
서버가 응답하면, 서버에서 데이터를 수신할 수 있다.  
그러기 위해서는 요구 핸들을 지정하고 **WinHttpQueryDataAvailable** 함수를 호출하여 다운로드 가능한 바이트 수를 취득한 뒤 **WinHttpReadData** 함수를 호출하여 데이터를 읽는다.  
요구가 동기적으로 이뤄질 경우 데이터가 이용 가능하게 될 때까지 WinHttpQueryDataAvailable 함수는 차단된다.  
  
  
### WinHttpSendRequest
  
``` 
WCHAR szHeader[] = L"Content-Type: application/x-www-form-urlencoded\r\n";
CHAR  szData[] = "msg=abc";
DWORD dwHeaderLength = lstrlenW(szHeader);
DWORD dwDataLength = lstrlenA(szData);

WinHttpSendRequest(hRequest, szHeader, dwHeaderLength, szData, dwDataLength, dwDataLength, 0);
```
4번째 인수 변수에 body에서 포함하고자 하는 데이터를 지정하고,  
5번째 인수에 이 데이터의 사이즈를 지정한다.  
6번째 변수는 데이터의 총 크기이지만, 나중에 WinHttpWriteData를 호출할 생각이 없는 경우는 5번째 인수의 값과 동일해도 상관없다.  
  데이터 "msg=abc" 이것은 msg에 abc라는 값을 설정한다는 의미이다.  
2번째 인수는 HTTP 헤더에 추가할 문자열이며, POST를 실행하는 경우는 위의 문자열을 지정하는 것이 일반적이다.  
이것에 의해서 서버는 데이터가 어떤 형식으로 전달됐는지를 이해할 수 있다.
  
  
### WinHttpWriteData
서버에 송신하고자 하는 데이터의 사이즈가 큰 같은 경우는 WinHttpSendRequest에 데이터 모두를 지정하지 않는 편이 좋을 수 있다.  
이런 경우는 데이터를 여러 개로 분할하고, WinHttpWriteData를 반복하여 호출 하는 방법도 있다.  
  
```
BOOL WINAPI WinHttpWriteData( HINTERNET hRequest, 
                    LPCVOID lpBuffer, 
                    DWORD dwNumberOfBytesToWrite, 
                    LPDWORD lpdwNumberOfBytesWritten);
```
hRequest는 리퀘스트 핸들을 지정한다.  
lpBuffer는 발송하고자 하는 데이터를 저장한 버퍼를 지정한다.  
dwNumberOfBytesToWrite는 발송하고자 하는 데이터의 사이즈를 지정한다.  
lpdwNumberOfBytesWritten는 발송된 데이터의 사이즈를 받을 변수의 주소를 지정한다.  
  
  
### WinHttpOpenRequest
POST 요청을 발행할 경우는 WinHttpOpenRequest의 제2 인수로 "POST"를 지정하도록 한다.  
WinHttpSendRequest의 4번째 매개 변수에 데이터를 지정할 수 있지만 이번에는 WinHttpWriteData에서 데이터를 보내도록 WINHTTP_NO_REQUEST_DATA를 지정한다.  
이 경우 WinHttpSendRequest는 HTTP 헤더만을 송신한다.   
제6 인수의 값은 WinHttpWriteData에서 송신하게 되는 데이터의 총 크기이다.  
WinHttpWriteData를 여러번 호출한 경우는 그 호출에 있어서의 제 3인수의 값을 모두 더한 것을 dwTotalLength에 지정한다.    
POST에서는 데이터를 분할해서 보낼 수 있는데 마지막 데이터를 보낸 것을 서버는 어떻게 알 수 있을까? 사실 이것은 WinHttpReceiveResponse가 하고 있다.  
이 함수는 서버에 보낼 데이터가 없음을 통보, 서버에서 응답을 받는다.  
참고로 GET의 경우는 WinHttpSendRequest 시점에서 응답이 오는데 이런 경우에도 WinHttpReceiveResponse를 호출하는 것이 기본이다.  
  
  
### WinHttpQueryHeaders
WinHttpQueryHeaders에서는 제2 인수에 WINHTTP_QUERY_STATUS_CODE을 지정하고 헤더 전체가 아닌 상태 코드만 얻는다.  
이것에 의해서, HTTP 요청의 성공 여부를 확인하기 쉽게 된다.   
```
dwSize = sizeof(DWORD);
WinHttpQueryHeaders(hRequest, WINHTTP_QUERY_STATUS_CODE | WINHTTP_QUERY_FLAG_NUMBER, WINHTTP_HEADER_NAME_BY_INDEX, &dwStatusCode, &dwSize, WINHTTP_NO_HEADER_INDEX);
if (dwStatusCode == HTTP_STATUS_OK) {
    WinHttpReadData(hRequest, buffer, sizeof(buffer), NULL);
    MessageBoxA(NULL, (LPSTR)buffer, "ボディ", MB_OK);
}
else {
    TCHAR szBuf[256];
    wsprintf(szBuf, TEXT("Status Code %d"), dwStatusCode);
    MessageBox(NULL, szBuf, NULL, MB_ICONWARNING);
}
```
제2 인수에 WINHTTP_QUERY_FLAG_NUMBER를 지정하고 있는 점도 중요하다.  
이로써 상태 코드를 DWORD 타입으로 받을 수 있다.  
값이 HTTP_STATUS_OK인 경우는 HTTP 요청이 성공했다는 것으로 WinHttpReadData에 의해 body를 얻는다.  
WinHttpQueryDataAvailable를 호출하여 데이터의 사이즈를 얻을 수 있지만 위 코드에서는 미리 큰 크기의 버퍼를 준비하는 방법을 사용하고 있다.
  
<br>  
  
[예제코드](/exampe_codes/ConsoleApplicationWin32_WinHttp.zip)  
    
<br>  
    
ps: 이 글들은 웹에서 본 글들을 정리한 것이다. 현재 출처가 기억나지 않아서 출처를 남기지 못하였다  