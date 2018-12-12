---
layout: post
title: golang - Go 애플리케이션을 Supervisor로 대몬화
published: true
categories: [Golang]
tags: go Supervisor daemon
---
[원문](https://qiita.com/gericass/items/fa794bfac5c6bd3e0aab)  
  
Go는 언어 레벨에서 daemon화가 지원되지 않으므로 Supervisor 이나 systemd로 daemon화 한다.  
  
## Supervisor 도입 

```
sudo apt-get install supervisor
```
  

## Go 애플리케이션 빌드

```
go build hoge
```
  
  
  
## supervisord.conf 편집

```
sudo vim /etc/supervisor/supervisord.conf
```  
  
아래 내용을 기록  
```
[program:hoge]
command = /path/to/hoge
autostart = true
startsecs = 5
user = root
redirect_stderr = true
```  
  
    
## supervisor 실행 

```
sudo supervisord
```  
  

  
### 에러가 나온다면

> Error: Another program is already listening on a port that one of our HTTP servers is configured to use. Shut this program down first before starting supervisord.  
  
위와 같은 것이 나온다면  
```
ps -ef | grep supervisord
```  
를 실행  
  
<pre>  
root     17310     1  0 05:40 ?        00:00:02 /usr/bin/python /usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf
pi       17638 16611  0 06:17 pts/0    00:00:00 grep --color=auto supervisord
</pre>  

pid를 확인하고 kill 한다  
```
kill 17310
```  
  