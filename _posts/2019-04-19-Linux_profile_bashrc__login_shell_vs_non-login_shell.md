---
layout: post
title: Linux - 리눅스 profile, bashrc 와 login shell vs non-login shell 의 이해
published: true
categories: [OS]
tags: Linux OS shell bashrc bashrc
---
[출처](http://webdir.tistory.com/126)  
  
## Shell 목록 보기
사용가능한 shell 목록 보기  
```
cat /etc/shells

  /bin/sh
  /bin/bash
  /sbin/nologin
  /bin/dash  
```
  
현재 사용중인 shell의 확인  
```
echo $SHELL

  /bin/bash
```
  
  
  
## Login shell vs Non-login shell
로그인 shell은 사용자가 로그인 했을때 적용되는 shell을 의미한다(default bash).  
로그인하여 bash가 처음 시작할때(login shell 일때), 다음 스크립트 파일들을 수행하여 환경을 설정한다.  
  
1. /etc/profile  
2. ~/.bash_profile    or    ~/.bash_login    or    ~/.profile  
3. ~/.bashrc  
4. /etc/bashrc  
  
/etc/profile 과 .profile은 shell이 bash가 아니라도 로그인하면 로드되어 적용되고 .bashrc 와 .bash_login, .bash_profile은 bash shell로 로그인 되었을 경우만 적용이 된다.  
  
로그인 shell이 로그아웃할때 다음 파일을 찾아서 수행한다.  
<pre>  
~/.bash_logout
</pre>
  
비 로그인 shell은 로그인 없이 실행되는 shell을 말한다. GUI 환경에서 새 터미널을 여는 경우가 이에 해당하며 sudo bash 나 su 같은 것도 이에 해당한다.  
- /etc/profile 과 /etc/bashrc 은 전체 사용자에게 적용된다.  
- ~/.bashrc 와 ~/.profile, ~/.bash_profile 등은 해당 사용자에게만 적용된다.   
    
사용자가 로그인을 하면 /etc/profile 과 /home/사용자/.profile 의 스크립트가 작동하여 적용된다. 이는 도스에서 autoexec.bat 파일과 같은 맥락이다.  



