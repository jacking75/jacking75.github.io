---
layout: post
title: C++ - Windows 텍스트 인코딩 변환
published: true
categories: [C++]
tags: c++ string uft-8 ansi mbcs wbcs encoding wchar_t
---
## Ansi <-> UTF-8
  
```
std::string CGlobal::ToUFT8( const char* pszIn )
{
    std::string resultString;

    int nLenOfUni = 0, nLenOfUTF = 0;
    wchar_t* us = NULL;
    char* pszOut = NULL;

    if ( ( nLenOfUni = MultiByteToWideChar( CP_ACP, 0, pszIn, strlen(pszIn), NULL, 0)) <= 0 )
        return 0;

    us = new wchar_t[nLenOfUni+1];
    memset(us, 0x00, sizeof(wchar_t)*(nLenOfUni+1) );

    // ansi ---> unicode
    nLenOfUni = MultiByteToWideChar( CP_ACP, 0, pszIn, strlen(pszIn), us, nLenOfUni );

    if( (nLenOfUTF = WideCharToMultiByte( CP_UTF8, 0, us, nLenOfUni, NULL, 0, NULL, NULL)) <= 0 )
    {
        //free(us);
        delete[] us;
        return 0;
    }

    pszOut = new char[nLenOfUTF+1];
    memset(pszOut, 0x00, sizeof(char)*(nLenOfUTF+1) );

    // unicode ---> utf8
    nLenOfUTF = WideCharToMultiByte(CP_UTF8, 0, us, nLenOfUni, pszOut, nLenOfUTF, NULL, NULL );
    pszOut[nLenOfUTF] = 0;
    resultString = pszOut;

    delete[] us;
    delete[] pszOut;

    return resultString;

}

std::string CGlobal::ToAnsi( const char* pszIn )
{
    std::string resultString;

    int nLenOfUni = 0, nLenOfANSI = 0;
    wchar_t* us = NULL;
    char* pszOut = NULL;

    if ( ( nLenOfUni = MultiByteToWideChar( CP_UTF8, 0, pszIn, strlen(pszIn), NULL, 0)) <= 0 )
        return 0;

    us = new wchar_t[nLenOfUni+1];
    memset(us, 0x00, sizeof(wchar_t)*(nLenOfUni+1) );

    // utf8 --> unicode
    nLenOfUni = MultiByteToWideChar( CP_UTF8, 0, pszIn, strlen(pszIn), us, nLenOfUni );

    if( (nLenOfANSI = WideCharToMultiByte( CP_ACP, 0, us, nLenOfUni, NULL, 0, NULL, NULL)) <= 0 )
    {
        //free(us);
        delete[] us;
        return 0;
    }

    pszOut = new char[nLenOfANSI+1];
    memset(pszOut, 0x00, sizeof(char)*(nLenOfANSI+1) );

    // unicode --> ansi
    nLenOfANSI = WideCharToMultiByte(CP_ACP, 0, us, nLenOfUni, pszOut, nLenOfANSI, NULL, NULL );
    pszOut[nLenOfANSI] = 0;
    resultString = pszOut;

    delete[] us;
    delete[] pszOut;

    return resultString;
}
```
  
  
  
## 가장 간단한 방법
코드 변화 방법이 아닌 sprintf 문을 사용하여 변환이 가능하다. 그리고 꼭 setlocals 함수로 문자 코드표를 지정해야 한다.  
사용자 컴퓨터의 OS에 따라간다면 setlocale(LC_ALL,"");  필요한 헤더 파일  <locale.h>  
  
- Ansi -> UniCode
  
```
 _snwprintf_s를 사용한다. 핵심은 인자 중 유니코드는 %s를 사용하지만 Ansi인 경우는 %S를 사용한다.
```
  
```
_snwprintf_s( acBuffer, _countof(acBuffer), _TRUNCATE, L"[%s] %S", L"유니코드문자열", "안시코드문자열");
```
  
- UniCode -> Ansi
  
```
_snprintf_s를 사용한다. 핵심은 인자 중 유니코드는 %S를 사용하지만 Ansi인 경우는 %s를 사용한다.
```
  
```
_snprintf_s( acBuffer, _countof(acBuffer), _TRUNCATE, "[%S] %s", L"유니코드문자열", "안시코드문자열");
```
  
  
  
## C의 표준 함수 사용
- 출처 : http://micropilot.tistory.com/1601
  
```
#include <stdio.h>
#include <string.h>
#include <locale.h>
#include <stdlib.h>

/* MBCS(Multi Byte Character System)
*  WBCS(Wide Byte Character System)
*  MBCS문자열은 문자열 중에 영문은 1바이트, 한글등은 2바이트로 저장함
*  WBCS문자열은 모든 문자를 2바이트(Unicode)로 저장함
*  MBCS, WBCS상호 변환가능
*  mbstowcs(wchar_t *dest, const char *src, size_t maxCount)
*  wcstombs(char *dest, const wchar_t *src, size_t maxCount)
*/

int main(void) {

 char *mbs = "한글과 Elglish 혼용";
 wchar_t wcsarr[36];
 char chr[36];

 setlocale(LC_ALL, "korean");

 printf("mblen(mbs,strlen(mbs))=%d \n", mblen(mbs,strlen(mbs)));

 /* mbcs(multi byte character system) 문자열을 wbcs(wide byte character system)으로 변환 */
 if(mblen(mbs, strlen(mbs))==2) { // 문자열 중에 2바이트 문자(한글)가 있다면 2를 리턴함
  mbstowcs(wcsarr, mbs, strlen(mbs)); // mbcs문자열을 지정한 바이트 수만큼 wbcs문자열로 변환
  wprintf(L"%s \n", wcsarr); // wbcs문자열 출력
 }

 // wbcs문자열을 지정한 바이트 수만큼 mbcs 문자열로 변환
 wcstombs(chr,wcsarr, sizeof(wcsarr));
 printf("%s \n", chr);

}
```
  