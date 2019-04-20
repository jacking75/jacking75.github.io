---
layout: post
title: C++ - [펌] Linux에서 라이브러리 로딩
published: true
categories: [C++]
tags: c++ lib linux
---
출처: [라이브러리 로딩 - ld.so.conf](http://blog.daum.net/mzinboy/3)
  
  
## 라이브러리 로딩 - ld.so.conf
- 동적 라이브러리를 호출하기 위해서는 path 지정이 필수이다.  
- 해당 라이브러리가 어디에 위치해 있는지 모든 디렉토리를 탐색하기에는 비효율적이기 때문이다.  
- 우리가 흔히 설정하는 LD_LIBRARY_PATH 환경변수가 동적 라이브러리 호출을 위한 path 지정에 사용되는 환경 변수이며 또 다른 방법으로 시스템 설정을 통해서 지정할 수 있다.
- 이때 사용되는 설정 파일이 리눅스 기준으로 /etc/ld.so.conf 파일이 되겠다.
- 리눅스 Kernel 2.6 이후 부터 /etc/ld.so.conf.d 디렉토리에 "xxx.conf" 파일 형식으로 설정할 수 있다.
- 로더는 LD_LIBRARY_PATH 또는 ld.so.conf 에 명시된 디렉토리에서 동적 라이브러리를 찾으며 해당 디렉토리에 없으면 Not Found 에러를 출력한다.
- 컴파일 시 link 오류나 실행시 loading 오류를 만나게 된다면 위 두가지 설정을 점검해 보기 바란다.
- LD_LIBRARY_PATH 나 ld.so.conf 두군대 모두에게 설정될 필요는 없으나 다음과 같은 경우에는 ld.so.conf 에 설정하도록 유의해 보자.
- 바이너리에 setuid, setgid 등이 설정된 경우 리눅스의 로더는 LD_LIBRARY_PATH 환경 변수 설정을 무시해 버린다고 한다.
- 그도 그럴것이 악의적으로 시스템 주요 함수들이 setuid를 가진 바이너리가 호출한 라이브러리에 의해서 오버라이딩 된다면??? 끔직할 것이다.
- 그런 보안을 고려한 것인지 모르겠지만 로더는 ( uid != euid || gid != egid ) 상황에서 LD_LIBRARY_PATH 를 무시하기 때문에 아무리 컴파일시에 link가 정상이고 ldd로 확인해도 정상이지만 바이너리 실행시 해당 라이브러리를 찾을 수 없다는 오류를 만나게 될 것이다.
- 이런 경우는 LD_LIBRARY_PATH가 아니라 ld.so.conf 에 설정을 해줘야 한다.
  
  
### 명령어
  
- ldconfig
    - /etc/ld.so.conf 설정된 동적 라이브러리 정보를 /etc/ld.so.cache 파일로 만들어 주는 일을 한다. 이로서 로더는 ld.so.cache 정보를 기반으로 보다 빠르게 라이브러리를 찾아 낼 수 가 있다. ld.so.conf 설정을 변경하면 반드시 ldconfig 명령을 수행하여 cache 를 갱신해 주도록 하자.
  
<pre>
ldconfig
ldconfig -p
</pre>
  
- ldd
    - 공유 라이브러리의 의존성을 검사해 준다. 동적 라이브러리도 공유 라이브러리 이므로 link 시나 load 시에 에러가 난다면 점검을 해보자.
  
<pre>
ldd <binary name>
</pre>
  
  
  
## 리눅스에서 무슨무슨 so 파일이 없다고 할 때
출처: [리눅스에서 무슨무슨 so 파일이 없다고 할 때](http://adnoctum.tistory.com/541)
  
### system default 경로
일반적으로 /usr/local/bin 과 /usr/bin 이다. 이 값은 /etc/ld.so.conf 파일에 설정이 된 값이다. 한 번 살펴 보자.  
즉, /etc/ld.so.conf 파일에 설정된 경로를 차례대로 돌아 가면서 원하는 so 파일을 찾는다. 물론, 만약 내가 어떤 프로그램을 만들었고, 그것을 내 프로그램이 설치된 경로 밑의 lib 경로에 넣고 그것을 ld.so.conf 에 넣고 싶으면, /etc/ld.so.conf.d/ 경로 밑에 xxx.conf 파일 이름으로 저장 후 그 파일에 그 경로를 집어 넣으면 된다는 것을 위 첫 번째 줄로부터 알 수 있다.  
즉, 첫 번째 줄이  
<pre>
include ld.so.conf.d/*.conf
</pre>  
이니까, ld.so.conf.d 경로에 있는 conf 로 끝나는 파일들을 살펴 보면,  
<pre>
[adnoctum@bioism ~]$ ll /etc/ld.so.conf.d/
 total 16
 -rw-r--r-- 1 root root 15 Sep  4  2009 mysql-i386.conf
 -rw-r--r-- 1 root root 20 Sep 14  2007 qt-i386.conf
 -rw-r--r-- 1 root root 25 Dec 10 08:44 xulrunner-32.conf
</pre>  
가 있고, 여기서 다시 mysql-i386.conf 파일의 내용을 보면,  
<pre>
[adnoctum@bioism ~]$ more /etc/ld.so.conf.d/mysql-i386.conf
 /usr/lib/mysql
</pre>
  
꼴랑 한 줄이다. 즉, include 구문에 의해, ld.so.conf.d 안에 있는 *.conf 파일의 내용들이 ld.so.conf 파일에 모두 들어 가게 된다.  
그냥 ld.so.conf 파일에 모두 넣지 왜 이렇게 하나 궁금할 수도 있는데, 만약 어떤 프로그램이 10 개의 경로에 so 파일을 분산시켜 넣는다고 하면 괜히 그것 때문에 ld.so.conf 파일이 지저분해 지게 되고, 또 다른 so 파일을 찾을 때 그것 때문에 여러 경로를 순회하므로 다른 프로그램들 성능까지 떨어 뜨리게 된다.  
만약 그렇게 지저분했던 프로그램을 지우면 더 골치아파지겠지. 저렇게 했다면 ld.so.conf.d 에 있던, 가령 myprogam.so.conf 파일 (이 파일이 10개의 경로를 갖고 있었다면) 만 지우면 간단히 해결되는 것이었음에도 불구하고, 그 내용이 모두 ld.so.conf 파일에 있었다면 그 경로를 지워도 되는지 (다른 so 파일이 거기 있었으면 지우면 안 될테니까) 아닌지 곤란해 지게 된다.  
하여튼, 저렇게 되어 있는 것이 깔끔하다.  
저렇게 수정을 했으면 적용하기 위해 ldconfig 명령어를 실행시켜 준다.  
  
  
### LD_LIBRARY_PATH
이 값은 환경 변수인데, 다음과 같이 직접 써 넣어 줄 수 있다.  
<pre>
~/.bash_profile 파일 내용중 일부.
</pre>
  
물론 이렇게 한 후 그것을 적용시키기 위해 source 를 해줘야 겠지.  
<pre>
[adnoctum@bioism ~]$ source ~/.bash_profile
</pre>
  
그런데 이런 방법은 그리 권장되는 것은 아닌듯 싶다.
  
  
  
### binary code에 hard-coding 된 경로
이 경우는 프로그래머가 아예 소스 코드에 so 파일 경로를 설정해 버린 경우인데, 실제의 예는, 내가 동적 라이브러리를 설명한 글에서 가져 온 다음의 코드이다.   
<pre>
const char* lib_path[] = { "/home/adnoctum/test/libabcd.so.1.0", "/home/adnoctum/test/libabcc.so.1.0"};
</pre>
  
그런데 이런 경우는 별로 없을 것이다. 또한 이런 경우 문제가 생기면 사용자가 해결할 수 있는 유일한 방법은 그 경로에 so 파일을 가져다 놓는 것인데, 프로그래머가 hard-coding 한 경로를 알지 못하면 도무지 방법이 없겠지. 하여튼 이런 경우는 거의 없다.  
위와 같이 설정된 경로에서 찾지 못했다고 한 so 파일은 그렇다면 어디에 있는가? 일단 locate 명령어로 찾아 본다. 그런데 아직 index 가 update 되지 않았을지도 모르므로 root 권한으로 updatedb 를 해주고 나서 locate 를 해준다.  
(물론 위 결과는 실제 의미 없다. 저 위 에러 메세지도 나에게서 난 것은 아니다. 그냥 구글링 결과에서 가져 왔을 뿐이다)  
  
그렇게 해서 원하는 so 파일을 찾았으면 /etd/ld.so.conf 파일에 그 so 파일이 있는 경로를 집어 넣거나, 아니면 so 파일을 /usr/local/bin 과 같은 곳으로 이동시키거나 link 를 만들어 준다. 그런데 일반적으로 프로그램은 여러 개의 so 파일이 사용하게 되므로 하나 옮기거나 링크 만들어도 또 다른 so 파일에서 동일한 에러가 날 수 있으므로 ld.so.conf 파일에 그 경로를 집어 넣는 것이 좋아 보인다. 혹은 LD_LIBRARY_PATH 에 그 경로를 덧붙이거나.  
그런데 locate 로 했는데 정말로 원하는 so 파일이 없는 것이라면, 마지막 방법으로 구글링을 해본다. 그러면 그 so 파일이 어떤 라이브러리에 있는지, 그래서 어떤 라이브러리를 설치해야 하는지를 알 수 있다.   
