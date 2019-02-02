---
layout: post
title: C++ - 안전한 문자열 조작 함수
published: true
categories: [C++]
tags: c++ string secure
---
### 문자열 복사

```
strcpy_s,  strcpy, _tcscpy_s,  wcscpy,  wcscpy_s
```
  
대체 추천 함수  
```
strncpy_s(char), _tcsncpy_s(TCHAR) ,   wcsncpy_s(wchar_t)

errno_t wcsncpy_s( wchar_t *strDest, strDest의 크기, const wchar_t *strSource, 복사할 문자 수 );

wchar_t szDest[256] = {0,};
wchar_t szSource[] = L"TEST";


wcsncpy_s( szDest, 256, szSource, 4 );

// or
// 널문자를 위해. szSource의 널문자까지만 복사. 255문자를 복사하는 것이 아님
wcsncpy_s( szDest, 256, szSource, 256-1 );
```
  
- 참고 [[http://gpgstudy.com/forum/viewtopic.php?p=97501]]
  
  
  
### 형식화된 문자열 만들기

```
printf_s,  _tprintf_s,  wprintf_s,  sprintf_s,  sprintf,  _stprintf_s,  swprintf_s,  _swprintf_s_l
```
  
대체 추천 함수  
```
_snprintf_s(char),  _sntprintf_s(TCHAR),  _snwprintf_s(wchar_t)
```
  
  
### 문자열 합치기

```
strcat_s,  strcat, _tcscat_s, wcscat_s
```
  
대체 추천 함수  
```
strncat_s(char),  _tcsncat_s (TCHAR),  wcsncat_s(wchar_t)
```
  
  
### 문자열 비교

```
strcmp,  _tcscmp,  wcscmp
```
대체 추천 함수
  
```  
strncmp(char),  _tcsnccmp(TCHAR) ,  wcsncmp(wchar_t)
```
  
  
### 문자열 길이 구하기

```
strlen,  strlen_s,  _tcsnlen,  wcsnlen
```
  
대체 추천 함수  
```
strnlen_s(char),  _tcsnlen(TCHAR) ,  wcsnlen_s(wchar_t)
```
  
  
### 가변 길이 문자열 만들기

```
vsprintf_s
```
  
대체 추천 함수  
```
_vsnprintf(char) ,  _vsntprintf TCHAR),  _vsnwprintf(wchar_t)
```
  
  
  
### StringCchCopy와 StringCbCopy

- StringCchCopy  두 번째 인수: 버퍼의 크기. 문자 단위
- StringCbCopy   두 번째 인수: 버퍼의 크기. 바이트 길이
- http://msdn.microsoft.com/en-us/library/windows/desktop/ms647527(v=vs.85).aspx
- 이미 위에 대체 함수가 있으므로 비추
  
  
  
###MultiByteToWideChar 사용 예

```
#include <windows.h>

int main()
{
	char str[100] = "삼국지천";
	int npwstrLen = strlen(str) + 1;

	// n1은 4, 널문자 포함 하지 않음
	wchar_t* pwstr1 = new wchar_t[npwstrLen];
	int n1 = MultiByteToWideChar(CP_ACP, 0, str, 8, pwstr1, npwstrLen);


	// n2는 5, 널문자 포함. 네 번째 인수를 -1 을 해야 널문자가 들어감
	wchar_t* pwstr2 = new wchar_t[npwstrLen];
	int n2 = MultiByteToWideChar(CP_ACP, 0, str, -1, pwstr2, npwstrLen);


	// n3는 4, 복사는 "삼국지" 까지 널문자 포함 하지 않음
	wchar_t* pwstr3 = new wchar_t[npwstrLen];
	int n3 = MultiByteToWideChar(CP_ACP, 0, str, 7, pwstr3, npwstrLen);


	delete[] pwstr1;
	delete[] pwstr2;
	delete[] pwstr3;

	return 0;
}
```
  
