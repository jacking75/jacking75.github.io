---
layout: post
title: Linux - PATH에 대해서
published: true
categories: [OS]
tags: OS Linux path
---
[출처](https://qiita.com/ryouya3948/items/8edbd5d744c83dd41141 )  
    
## 명령 검색 PATH의 확인 방법
`$ echo $PATH`로 확인 가능하다.  
<pre>
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/sbin 
</pre>
  
경로는 `:`로 구분되고, /usr/local/bin, /usr/bin, /bin, /usr/sbin, /sbin, /usr/local/sbin 6개가 검색 명령 경로로 설정되어 있다.  
  
  
## 명령 실행 파일의 저장 위치 확인 방법
`$ which ls`로 확인할 수 있다. 
  
  
## 동명의 실행 파일이 여러 탐색 명령 경로에 있을 때
이 경우 우선 순위가 있고, `$ echo $PATH`에서 출력된 왼쪽에서부터 실행된다.  
예를 들어 아래와 같은 경우라면  
<pre>
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/sbin
</pre>
  
/usr/local/bin 에서 실행된다.  
우선 순위로는 /usr/local/bin > /usr/bin > /bin > /usr/sbin > /sbin > /usr/local/sbin 이다.  
  
  
## PATH 추가
PATH를 추가하려면 .bashrc 또는 .bash_profile 파일에 `export PATH=$PATH:추가 할 명령 검색 경로` 형식으로 추가한다.  
  
작성한 파일을 source 명령을 실행하지 않으면 경로가 통하지 않는다.  
<pre>
$ source ~/.bashrc
혹은
$ source ~/.bash_profile
</pre>
  
  
  
## export 명령?
환경 변수를 보거나 설정할 수 있다.  
  
## 환경 변수보기
  
```  
export -p
```
  
  
## 환경 변수 설정
예를 들어 `$ ULB`라는 환경 변수를 설정하려면 아래와 같이 설정한다.  
<pre>
$ export ULB=/usr/local/bin

$ echo $ULB
/usr/local/bin #출력 결과
</pre>
  
  
`$ echo $ULB`로 설정되어 있는지 확인한다.  
<pre>
ls / usr / local / bin
ls $ ULB
</pre>
  
두 명령 모두 같은 결과가 된다.  
  
  
## 환경 변수 재정의
  
```
$ echo $ULB
/usr/local/bin #출력 결과

$ export ULB=/u/l/b

$ echo $ULB
/u/l/b #출력 결과
```
  
  
  
## 환경 변수 삭제
`unset` 명령으로 제거 할 수 있다.  
  
  
## 추가한 명령 탐색 경로의 우선 순위를 높게하기
  
```
$ export PATH=추가 할 명령 검색 경로:$PATH
```
  
명령 탐색 경로는 왼쪽에서 우선 순위가 높아지고 있기 때문에, 이렇게 기재한다.  
  