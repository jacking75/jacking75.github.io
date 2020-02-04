---
layout: post
title: Linux How To
published: true
categories: [OS]
tags: os linux howto
---
# 네트워크
  
## 특정 port에 접속한 네트워크 정보 보기
32452 port에 접속한 네트워크 정보를 보고 싶은 경우   
```
netstat -nap | grep :32452
```
  
  
# 파일
디렉토리 이름 변경   
<pre>
mv 옵션... 원본 대상
mv 옵션... 원본... 디렉토리
mv 옵션... 디렉토리 디렉토리
</pre>  
  
파일 path 추가
- home/본인아이디 로 이동
- `.bashrc` 파일열고 아래 내용 추가
- `export PATH=$PATH:/home/본인아이디/추가디렉토리지정`   
    
     
    
# tar.bz2, tar.gz 파일 압축 풀기 명령어
tar.bz2 파일 압축 풀기    
```  
tar -xvf test.tar.bz2
```
  
  
tar.bz2 압축 내용 보기  
```
tar -tvf test.tar.bz2
```
  
  
tar.gz 파일 압축 풀기  
```
tar -xvzf test.tar.gz
```
  

# 시스템 종료와 재부팅
shutdown 옵션 시간 메시지    
-t n : 경고 메세지를 보낸 후 n초 후에 kill 시그널을 보낸다.  
-h : shutdown시 halt를 실행하게 한다.  
-n : 디스크 동기화 동작의 수행을 금지한다.  
-r : 시스템을 재부팅한다.  
-f : 다음 부팅시 파일시스템 검사를 하지 않는다.  
-c : 이미 예약되어 있는 shutdown을 취소한다. 이 옵션을 둔다면 시간인수는 줄 수 없다. 하지만 메시지는 사용자들에게 줄 수 있다.    
-k : 모든 동작을 제대로 수행하지만, 실제로 시스템을 종료하지는 않는다.  
  
시스템 종료시 가장 자주 사용되는 방식 : shutdown -h now  
시스템 재부팅시 자장 자주 사용되는 방식 : shutdown -r now  
시스템 종료 예약 : shutdown -h 10 (10분 후에 시스템을 종료한다.)  
     
    
    
# 우분투 PPA
- https://webdir.tistory.com/197
- https://launchpad.net/ubuntu    
  
  
  
  
