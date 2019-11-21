---
layout: post
title: C++17 - 숫자to문자열, 문자열to숫자 변환 표준 라이브러리
published: true
categories: [C++]
tags: c++ c++17 charconv from_chars to_chars
---
C++17에서 숫자를 문자열로 변환, 문자열을 숫자로 변환 해주는 라이브러리가 새로 생김  
[출처](https://speakerdeck.com/kariyamitsuru/new-features-of-c-plus-plus-17-gleaner )  
  
  
## 특징
- 서식 지정은 문자열이 아니다
- 동적 메모리 할당을 하지 않는다
- 로케일 무시
- 가상 함수 호출이 아니다
- 버퍼 오버런 방지
- 반환 값에서 에러를 반환한다
- 입력 문자열의 빈 공간은 무시하지 않는다
- 선두의 0x는 해석 하지 않는다.
  
  
  
## 헤더 파일
  
```
#include <charconv>
```
  
  
  
## 문자열에서 숫자로 변환
  
```
std::from_chars_result from_chars(const char* first, const char* last, /*see below*/& value, int base = 10);
std::from_chars_result from_chars(const char* first, const char* last, float& value, std::chars_format fmt = std::chars_format::general);
std::from_chars_result from_chars(const char* first, const char* last, double& value, std::chars_format fmt = std::chars_format::general);
std::from_chars_result from_chars(const char* first, const char* last, long double& value, std::chars_format fmt = std::chars_format::general);

struct from_chars_result {
    const char* ptr; // 숫자 값으로 해석 할 수 없는 문자로의 포인터
    std::errc ec; // 변환 실패한 경우의 에러 코드
};

enum class chars_format {
    scientific = /*unspecified*/, // %e와 같음
    fixed = /*unspecified*/,      // %f와 같음
    hex = /*unspecified*/,        // %a와 거의 같음
    general = fixed | scientific   // %g와 같음
};
```
  
  
### 사용 예
  
```
const char s[] = "1975s";
int i = 0;
auto r = std::from_chars(s, s + sizeof(s) - 1, i);

assert(i == 42);
assert(r.ptr == s + 4); 
assert(r.ec == std::errc{});
```
  
r.ptr은 숫자로 해석할 수 없는 문자(위 코드에서는 s)를 가리킨다.
  
  
#### 마이너스 등호 사용  
  
```
const char s[] = "-1975s";
int i = 0;
auto r = std::from_chars(s, s + sizeof(s) - 1, i);

assert(i == -1975);
assert(r.ptr == s + 5); 
assert(r.ec == std::errc{});
```
  
  
#### 플러스 등호는 사용 불가
  
```
const char s[] = "+1975s";
int i = 114;
auto r = std::from_chars(s, s + sizeof(s) - 1, i);

assert(i == 114);
assert(r.ptr == s); 
assert(r.ec == std::errc::invalid_argument);
```
  
  
#### 숫자가 너무 큰 경우 에러
  
```
const char s[] = "12345678901234567890";
int i = 114;
auto r = std::from_chars(s, s + sizeof(s) - 1, i);

assert(i == 114);
assert(r.ptr == s + sizeof(s) - 1); 
assert(r.ec == std::errc::result_out_of_range);
```
  
이 경우도 r.ptr은 숫자로 해석할 수 없는 문자를 가리키므로 최후의 숫자 다음의 문자(즉 종단)를 가리키고 있다.  
  
  
#### 16진수는 가능
  
```  
const char s[] = "DEADBEEF";
int i = 42;
auto r = std::from_chars(s, s + sizeof(s) - 1, i, 16);

assert(i == 0xDEAD'BEEF);
assert(r.ptr == s + 8); 
assert(r.ec == std::errc{});
```  
  
선두에 0x 가 붙으면 에러  
```  
const char s[] = "0xDEADBEEF";
int i = 42;
auto r = std::from_chars(s, s + sizeof(s) - 1, i, 16);

assert(i == 0);
assert(r.ptr == s1); 
assert(r.ec == std::errc{});
```  
에러는 아니지만 선두의 0만 변환 된다.  
  
  
#### 부동 소수정
  
```  
const char s[] = "42.195km";
double d = 0;
auto r = std::from_chars(s, s + sizeof(s) - 1, d);

assert(i == 42.195);
assert(r.ptr == s + 6); 
assert(r.ec == std::errc{});
```  
  
**앞의 정수와 같이 선두에 + 등호가 붙으면 에러**    
  
지수부의 + 등호는 괜찮음  
```  
const char s[] = "2.99792458e+8m/s";
double d = 0;
auto r = std::from_chars(s, s + sizeof(s) - 1, d);

assert(i == 2.997'924'58e+8);
assert(r.ptr == s + 13); 
assert(r.ec == std::errc{});
```  
  
fmt가 scientific인 경우는 지부가 없으면 에러  
```
const char s[] = "299792458m/s";
double d = 0;
auto r = std::from_chars(s, s + sizeof(s) - 1, d, std::chars_format::scientific);

assert(i == 0);
assert(r.ptr == s); 
assert(r.ec == std::errc::invalid_argument);
```
  
  
  
## 숫자에서 문자열로 변환
  
```
std::to_chars_result to_chars(char* first, char* last, /*see below*/ value, int base = 10);
std::to_chars_result to_chars(char* first, char* last, float       value);
std::to_chars_result to_chars(char* first, char* last, double      value);
std::to_chars_result to_chars(char* first, char* last, long double value);
std::to_chars_result to_chars(char* first, char* last, float value,  std::chars_format fmt);
std::to_chars_result to_chars(char* first, char* last, double value, std::chars_format fmt);
std::to_chars_result to_chars(char* first, char* last, long double value, std::chars_format fmt);
std::to_chars_result to_chars(char* first, char* last, float value, std::chars_format fmt, int precision);
std::to_chars_result to_chars(char* first, char* last, double value, std::chars_format fmt, int precision);
std::to_chars_result to_chars(char* first, char* last, long double value, std::chars_format fmt, int precision);

struct to_chars_result {
    char* ptr;    //출력된 최후의 문자 다음의 포인터
    std::errc ec;  // 변환 실패한 경우의 에러 코드
};
```  
  
  
### 사용 예 
  
```
int i = 42;
char s[10];
auto r = std::to_chars(s, s + sizeof(s), i);

assert("42"sv == std::string_view(s, r.ptr - s));
assert(r.ec == std::errc{});
```  
출력은 '\0'으로 종단 되지 않으므로 주의  
  
  
#### 16진수 
  
```
int i = 42;
char s[10];
auto r = std::to_chars(s, s + sizeof(s), i, 16);

assert("2a"sv == std::string_view(s, r.ptr - s));
assert(r.ec == std::errc{});
```  
선두에 0x 는 붙지 않는다. 영문자 출력은 소문자(대문자 버전은 없다).  
  
  
#### 버퍼가 부족하면 에러
  
```
int i = 114'514;
char s[5];
auto r = std::to_chars(s, s + sizeof(s), i);

assert(r.ptr == s + sizeof(s));
assert(r.ec == std::errc::value_too_large);
```  
변환 에러의 경우 ptr은 마지막을 가리킨다. 변환 에러의 경우 버퍼 안이 어떻게 될지는 미정의.  
  
  
#### 부동 소수점
  
```
double d = 100'000;
char s[10];
auto r = std::to_chars(s, s + sizeof(s), d);

assert(r.ec == std::errc{});
assert("1e+05"sv == std::string_view(s, r.ptr - s));
```  
포맷을 지정하지 않으면 로케일 C의 printf에서 %f와 %e로 변환한 경우에 문자열이 짧아지게 된다.  
같은 길이의 경우에는 %f를 우선.  
printf의 %g와는 다르므로 주의.  
  
  
#### 정밀도 지정  
  
```
double d1 = 3.1415926653589793238462643383279;
char s[50];
auto r = std::to_chars(s, s + sizeof(s), d1);
assert(r.ec == std::errc{});

double d2 = 0;
auto r2 = std::to_chars(s, r1.ptr, d2);
assert(d1 == d2)
assert(r.ec == std::errc{});
```    
부동 소수점은 문자열 화 할 때 정밀도를 지정하지 않은 경우 라운드트립 해서 결과가 같아 지도록 처리된다.  
포맷을 지정한 경우도 같다.  
  
  
```
double d = 3.1415926653589793238462643383279;
char s[100];
auto r = std::to_chars(s, s + sizeof(s), d, std::chars_format::general, 3);

assert(r.ec == std::errc{});
assert("3.14"sv == std::string_view(s, r.ptr - s));
```  
정밀도의 지정 내용은 printf의 대응하는 서식과 같다.    
  
  