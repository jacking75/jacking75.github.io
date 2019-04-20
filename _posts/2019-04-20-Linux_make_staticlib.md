---
layout: post
title: C++ - [펌] Linux에서 정적 라이브러리 만들기
published: true
categories: [C++]
tags: c++ staticlib linux
---
출처: [리눅스 정적 라이브러리 만들어 쓰기|작성자 라온이아부지 선성태] (http://blog.naver.com/r2adne/120127876141)  
  
  
## 순서
- 라이브러리로 쓸 함수가 포함된 파일 두개 만들기
- 컴파일 해서 오브젝트로 만들기
- 두개의 오브젝트를 하나의 라이브러리로 합치기
- 라이브러리를 사용할 메인프로그램 코딩
- 라이브러리를 포함해서 컴파일
- 동작 실행
  
  
  
## 라이브러리로 쓸 함수가 포함된 파일 두개 만들기
  
```
//file1.c
#include <stdio.h>


void test1()
{
 printf(" test 1 \n");
}


void test2()
{
 printf(" test 2 \n");
}
```
  
```
//file2.c
#include <stdio.h>

void test3()
{
 printf(" test 3 \n");
}
```
  
두 파일에 포함된 함수들은 단순히 호출되면 일정문구를 출력 하는 기능을 하지만, 라이브러리 동작 테스트 하기에는 큰 무리가 없어 보인다.  
  
  
  
## 컴파일 해서 오브젝트로 만들기
  
<pre>
#gcc -c file1.c file2.c
</pre>
  
c 옵션은 오브젝트 파일을 만드는 옵션이다. 오브젝트 파일은 .o 확장 자를 가지며, elf 바이너리 구조이다.  
  
<pre>
결과물 : file1.o    file2.o
</pre>
  
  
  
## 두개의 오브젝트를 하나의 라이브러리로 합치기
  
<pre>
#ar rscv libstatic.a file1.o file2.o
</pre>
  
- 두개의 오브젝트 파일을 libstatic.a 파일로 합친다.
- ar은 tar 과 비슷하며, archive 를 뜻한다.
- 옵션 rscv 에 대한 설명이 자세히 나온곳은 아무데도 없는것 같고. ar 옵션을 검색해 보니 몇가지 나오긴 했다.
- ar 옵션
    - t : library 내용보기
    - p : library 소스보기
    - r : library insert or replace
    - s : Index  생성
    - x : 묶은 파일 풀기
    - d : 삭제
  
<pre>
결과물 : libstatic.a
</pre>
  
  
  
### 라이브러리를 사용할 메인프로그램 코딩
  
```
//test.c
#include <stdio.h>
#include "mylib.h"

int main()
{
 test1();
 test2();
 test3();

 return 0;
}
```
  
많은 사람들이 이와 비슷한 소스를 가지고 테스트를 하는듯 하며, 정확한 출처는 한빛미디어에서 나온 '유닉스.리눅스 프로그래밍 필수 유틸리티' 라는 책임을 밝힌다. 
  
  
  
## 라이브러리를 포함해서 컴파일
  
<pre>
#gcc -o test test.c -L./ -lstatic
</pre>
  
동적라이브러리 작성시 먼저 설명한 부분이지만... 마지막 -l라이브러리이름 부분은 반드시 lib이름 형태로 파일명을 작성하고, 이부분에 lib 를 ㅃ개 -l 을 넣어야 정확히 동작된다.  
  
<pre>
결과물 : 실행파일
</pre>
  
  
  
## 동작
  
<pre>
#./test
  test 1
  test 2
  test 3
#
</pre>
  
  
  

