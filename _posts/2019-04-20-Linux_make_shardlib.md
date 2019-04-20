---
layout: post
title: C++ - [펌] Linux에서 동적 라이브러리 만들기
published: true
categories: [C++]
tags: c++ sharedlib linux
---
출처: [리눅스 공유라이브러리(동적 라이브러리) 만들어 쓰기|작성자 라온이아부지 선성태](http://blog.naver.com/r2adne/120127832516)
  
  
## 순서
- 라이브러리로 쓸 함수가 포함된 소스 두 개 작성
- 공용으로 각각 컴파일
- 두개의 오브젝트를 하나의 라이브러리로 통합
- 링크파일 생성
- 라이브러리 등록
- 라이브러리의 함수를 사용 하는 프로그램 소스 작성
- 라이브러리를 사용하도록 컴파일
- 실행
  
  
  
## 라이브러리로 쓸 함수가 포함된 두개의 소스 작성
  
```
// file1.c
#include <stdio.h>


int add(int a, int b)
{
 int result=0;
 printf("add :  %d + %d = ",a,b);
 return a+b;
}

int sub(int a, int b)
{
 printf("sub :  %d - %d = ",a,b);
 return a-b;
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
  
  
  
## 두 파일 컴파일
  
<pre>
#gcc -fPIC -c file1.c file2.c
</pre>
  
-fPIC 옵션 은 위치 독립적인코드를 생성 하는 옵션이다.
    - fpic (소문자) 는 플랫폼에 제약이 생기지만 빠르고 가볍다.
    - fPIC (대문자) 는 플랫폼에 제약이 없지만 무겁고 느리다.  - 대부분 이걸 사용 한다.
- c 옵션: 오브젝트 파일을 생성 한다.
  
<pre>
결과물 : file1.o  file2.o
</pre>
  
  
  
## 두개의 오브젝트를 하나의 라이브러리로 통합
  
<pre>
#gcc -shared -Wl,-soname,libmy.so.0 -o libmy.so.0.0.0 file1.o file2.o
</pre>
  
- shared 는 공유 라이브러리를 사용해서 링크 하라는 옵션이다.
- Wl 는 뒤에 오는 옵션들을 링커(collect2) 에게 gcc 를 거치지 않고 바로 전달 하라는 옵션이다.
- soname 은 실제 만들 라이브러리인 libmy.so.0.0.0 에 libmy.so.0 이라는 soname 을 생성하여 나중에 동적링크(/lib/ld-linux.so.2)가 공유 라이브러리를 찾을때 libmy.so.0.0.0 을 찾는게 아니라 soname 인  libmy.so.0 를 찾아 링크 하도록 한다. 버전 관리를 융통성 있게 하기 위함이며, 실제 gcc 가 링크 하는 파일명은 버전 정보가 없는 libmy.so 를 상요 하며, 이또한 libmy.so0.0.0 의 링크 파일로 만들어 사용 하는 것이다.  
  
<pre>
결과물 : libmy.so.0.0.0
</pre>
  
  
  
## 링크 파일 생성
  
<pre>
$ln -s libmy.so.0.0.0 libmy.so                  --> gcc 링크를 위한 파일 생성
$ln -s libmy.so.0.0.0 libmy.so.0               --> 동적 링크를 위한 파일 생성
</pre>
  
<pre>
결과물 libmy.so      libmy.so.0      ( 둘다 링크 파일임)
</pre>
  
  
  
## 라이브러리 등록
라이브러리를 등록 하는 방법은 여러가지가 있으나 가장 간단한 방법으로 한다.  
원칙적으로 사용자가 만들어쓰는 라이브러리는 /usr/local/lib 에 넣어 쓰는 것을 권장 하지만, 현재 위치를 그냥 명시 하는 방법을 쓴다.  
  
<pre>>
#vi /etc/ld.so.conf
include ld.so.conf.d/*.conf       --> 리눅스 커널 2.6 에서 방식이 바껴서. 파일 위치를 명시하는 .conf 파일을 만들어 주면 자동으로 잡힘..
/mnt/samba/shared_lib_test     --> 라이브러리 경로를 추가 하고 저장 한다..( 그러나, 이렇게 강제로 실제 경로를 넣어도 되더라.)
/etc/ld.so.cache 파일 갱신
#ldconfig
-> 실제 위치는 /sbin/ldconfig 이므로, 그냥 써서 안되면 /sbin/ldconfig 를 치면 된다.
</pre>
  
  
  
### 라이브러리의 함수를 사용 하는 프로그램 소스 작성
  
```
//mylib.h 파일
int add(int,int);
int add(int,int);
void test3(void);
```
  
```
// test.c 파일
#include <stdio.h>
#include "mylib.h"

int main(void)
{
 int result = 0;
 result = add(5,6);
 printf("%d\n",result);

 result = 0;
 result = sub(10,4);
 printf("%d\n",result);

 test3();
 return 0;
}
```
  
  
  
## 라이브러리를 사용하도록 컴파일
  
<pre>
#gcc -o test test.c -L./ -lmy
</pre>
  
여기서 주의할 것은 -lmy 옵션이다... 이것 때문에 하루 죙일 삽질 했다.. -l 옵션 뒤에 바로 붙는 my 는 앞서 생성한 라이브러리 이름중 lib 를 제외한 나머지이다. 반드시 라이브러리는 libname 형태로 만들고, 이 옵션에서 -lname 형태로 써야만 컴파일이 된다.  
만약 이 형태를 따르지 않고, 예를 들어 -ltest 라고 했다면.. 아래와 같은 에러를 볼것이다.  
  
<pre>
/usr/bin/ld: cannot find -ltest
collect2: ld returned 1 exit status
</pre>
  
<pre>
결과물 : 실행 파일  test
</pre>
  
  
  
## 실행 결과
  
<pre>
#./test
@localhost shared_lib_test]# ./test
add :  5 + 6 = 11
sub :  10 - 4 = 6
 test 3
</pre>
  
