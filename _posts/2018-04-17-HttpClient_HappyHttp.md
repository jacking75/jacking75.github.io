---
layout: post
title: C++ http 클라이언트 요청하기 - HappyHttp
published: true
categories: [Network]
tags: cpp http happyhttp
---
# HappyHttp
Visual Studio 2017에서 빌드하면 에러가 난다. 조금 수정이 필요하다.  
happyhttp.cpp 파일의 상단에 아래의 전처리를 선언한다.  
```
#define _WINSOCK_DEPRECATED_NO_WARNINGS 1
#define _CRT_SECURE_NO_WARNINGS 1
```  
  
## 개요
- 오픈 소스 라이브러리. [링크](https://github.com/Zintinio/HappyHTTP)  
    - 위에 언급된 수정한 버전은 [여기](https://github.com/jacking75/HappyHTTP) 에 있다.  
- 멀티 플랫폼 지원.
- **happyhttp.h 와 happyhttp.cpp 단 두개의 파일 프로젝트에 포함하면 된다.**
  
  
## 예제 - HappyHTTP를 사용한 HTTP Post Request

```C++
//HTTP 헤더 선언
const char* headers[] =
    {
        "Connection", "close",
        "Content-type", "application/x-www-form-urlencoded",
        "Accept", "text/plain",
        0
    };

//HTTP BODY 선언
const char* body = "answer=42&name=Bubba";

//연결할 대상을 지정하고 요청 결과에 따른 콜백 함수를 지정한다.
happyhttp::Connection conn("scumways.com", 80);
conn.setcallbacks(OnBegin, OnData, OnComplete, 0);

//POST 형식으로 웹에 요청한다.
conn.request("POST", "/happyhttp/test.php", headers, (const unsigned char*)body, strlen(body));

//요청 결과가 아직 처리되지 않았다면 계속해서 대기한다.
while (conn.outstanding())
    conn.pump();
```

요청이 처리되면 응답을 처리하기 위한 콜백 함수 OnData 함수가 실행된다.
```
void OnData(const happyhttp::Response* r, void* userdata, const unsigned char* data, int n)
{
    fwrite(data, 1, n, stdout);
    count += n;
}
```
  
  
### 예제 - HTTP Post로 JSON 데이터 보내기 

```
#include "happyhttp.h"
#include <cstdio>
#include <cstring>

#ifdef WIN32
#include <winsock2.h>
#endif // WIN32

#pragma comment(lib, "ws2_32")

int count = 0;

void OnBegin(const happyhttp::Response* r, void* userdata)
{
    printf("BEGIN (%d %s)\n", r->getstatus(), r->getreason());
    count = 0;
}

void OnData(const happyhttp::Response* r, void* userdata, const unsigned char* data, int n)
{
    fwrite(data, 1, n, stdout);
    count += n;
}

void OnComplete(const happyhttp::Response* r, void* userdata)
{
    printf("COMPLETE (%d bytes)\n", count);
}



void Test1()
{
    puts("-----------------Test1------------------------");
    // simple simple GET
    happyhttp::Connection conn("scumways.com", 80);
    conn.setcallbacks(OnBegin, OnData, OnComplete, 0);

    conn.request("GET", "/happyhttp/test.php", 0, 0, 0);

    while (conn.outstanding())
        conn.pump();
}



void Test2()
{
    puts("-----------------Test2------------------------");
    // POST using high-level request interface

    const char* headers[] =
    {
        "Connection", "close",
        "Content-type", "application/x-www-form-urlencoded",
        "Accept", "text/plain",
        0
    };

    const char* body = "answer=42&name=Bubba";

    happyhttp::Connection conn("scumways.com", 80);
    conn.setcallbacks(OnBegin, OnData, OnComplete, 0);
    conn.request("POST",
        "/happyhttp/test.php",
        headers,
        (const unsigned char*)body,
        strlen(body));

    while (conn.outstanding())
        conn.pump();
}

void Test3()
{
    puts("-----------------Test3------------------------");
    // POST example using lower-level interface

    const char* params = "answer=42&foo=bar";
    int l = strlen(params);

    happyhttp::Connection conn("scumways.com", 80);
    conn.setcallbacks(OnBegin, OnData, OnComplete, 0);

    conn.putrequest("POST", "/happyhttp/test.php");
    conn.putheader("Connection", "close");
    conn.putheader("Content-Length", l);
    conn.putheader("Content-type", "application/x-www-form-urlencoded");
    conn.putheader("Accept", "text/plain");
    conn.endheaders();
    conn.send((const unsigned char*)params, l);

    while (conn.outstanding())
        conn.pump();
}

void Test4()
{
    puts("-----------------Test4------------------------");
    // POST example using lower-level interface

    auto reqJsonData = R"(
          {
            "UserSeq": 1,
            "UserID": "jacking75",
            "UserPW": "123qwe"
          }
        )";
    
    const char* headers[] =
    {
        "Connection", "close",
        "Content-type", "application/json",
        "Accept", "text/plain",
        0
    };

    //const char* body = "answer=42&name=Bubba";

    happyhttp::Connection conn("localhost", 19000);
    conn.setcallbacks(OnBegin, OnData, OnComplete, 0);
    conn.request("POST",
        "/Request/Login",
        headers,
        (const unsigned char*)reqJsonData,
        strlen(reqJsonData));

    while (conn.outstanding())
        conn.pump();
}


int main(int argc, char* argv[])
{
#ifdef WIN32
    WSAData wsaData;
    int code = WSAStartup(MAKEWORD(1, 1), &wsaData);
    if (code != 0)
    {
        fprintf(stderr, "shite. %d\n", code);
        return 0;
    }
#endif //WIN32
    try
    {
        //Test1();
        //Test2();
        //Test3();

        Test4();
    }

    catch (happyhttp::Wobbly& e)
    {
        fprintf(stderr, "Exception:\n%s\n", e.what());
    }

#ifdef WIN32
    WSACleanup();
#endif // WIN32

    return 0;
}
```
  
<br>  
  
[예제코드](/exampe_codes/ConsoleApplicationHappyHttp.zip)   