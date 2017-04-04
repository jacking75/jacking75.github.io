---
layout: post
title: std::codecvt_utf8로 wstring을 utf-8 문자열로 변환 하기 
published: true
categories: [C++]
tags: c++ std::codecvt_utf8
---
std::wstring을 utf-8 문자열로 변환 하는 방법은 아래와 같다.  

```
std::wstring wStr = L"우하하";
std::wstring_convert<std::codecvt_utf8<wchar_t>, wchar_t> utf8Conv;
auto utf8Str = utf8Conv.to_bytes(wStr.c_str());
```
  
wchar_t의 크기는 windows, linux 각각 크기가 다르다. 그래서 만약 크기를 특정 길이로 고정하고 싶다면 wchar_t 대신 char16_t, char32_t를 사용하여 크기를 고정 할 수 있다.  
  
```
std::u16string utf16Str = u"우하하";
std::wstring_convert<std::codecvt_utf8<char16_t>, char16_t> utf8Conv;
auto utf8Str = utf8Conv.to_bytes(utf16Str.c_str());
```
  
위 코드는 gcc나 clang 에서는 문제가 없지만 Visual Studio 2015 Update 3 에서는 자체 버그에 의해서 링크 에러가 발생한다.  
Visual Studio 2017 에서는 문제가 없을 것이다.  
Visual Studio 2015 Update 3 에서 해결 방법은 아래와 같다.  
  
```
std::u16string utf16Str = u"우하하";
std::wstring_convert<std::codecvt_utf8<uint16_t>, uint16_t> utf8Conv;
auto utf8Str = utf8Conv.to_bytes(reinterpret_cast<const uint16_t*>(utf16Str.c_str()));
```
  
 
<br>  
<br>  


출처: http://qiita.com/benikabocha/items/1fc76b8cea404e9591cf

