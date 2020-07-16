---
layout: post
title: Linux TCP 서버와 관련된 커널 파라메터 튜닝
published: true
categories: [OS]
tags: os linux kernel tuning
---
[출처](https://qiita.com/ryuichi1208/items/3bb7a270fe650b2f7260 )  
  
커널 파라메터 설명은 아래 링크에서 참고  
linux/Documentation/sysctl/  

예를 들어 net 계열의 파라메터 설명을 보고 싶다는 아래에서 확인한다.  
linux/Documentation/sysctl/net.txt  
  
  
  
# TCP 서버용 tuning
  
## 기본
  
```
# 항구적 설정
$ vi /etc/sysctl.conf
$ sysctl -p
```   
  
  
## kernel
  
```
# 공유 메모리의 최대 사이즈. 서버 탑재 메모리(1GB)에 맞추어서 변경
kernel.shmmax      = 1073741824

# 시스템 전체의 공유 메모리 ・페이지의 최대 수
kernel.shmall      = 262144

# 시스템 전체의 프로세스 수의 상한
kernel.threads-max = 1060863

# kernel panic으로 자동 재시작
kernel.panic=30
```
  
  
## 파일 시스템 관련
  
```  
# 파일 시스템에서의 오픈 파일의 상한
fs.file-max = 1619870
```
  
  
## 가상 메모리
  
```
# OOM killer 발동한 경우는 kernel panic을 일으켜서 재 시작 시킨다
vm.panic_on_oom=1
```
  
  
## Network
  
```
# 네트워크 디바이스 별로 커널이 처리할 수 있도록 모아 두는 queue의 크기
net.core.netdev_max_backlog

# backlog 값의 hard limit
net.core.somaxconn = 4096

# 소켓 당 SYN을 받고 ACK을 받지 못한 상태의 커넥션 유지 가능 수
net.ipv4.tcp_max_syn_backlog = 16000

# NIC 에 대한 수신 패킷의 최대 큐잉 수
net.core.netdev_max_backlog = 16000

# TCP와 UDP 수신 버퍼의 default 사이즈와 최대 사이즈
net.core.rmem_max = 4194304

# TCP와 UDP 송신 버퍼의 default와 최대 사이즈
net.core.wmem_max = 4194304

# 데이터 수신 버퍼 사이즈
net.ipv4.tcp_rmem = 4096        87380   4194304

# FIN 타임아웃 시간
net.ipv4.tcp_fin_timeout = 12

# tcp의 SYN을 보내는 리트라이 횟수
net.ipv4.tcp_syn_retries = 2

# TCP/IP 송신 시에 사용하는 포트의 범위
net.ipv4.ip_local_port_range = 32768    60999

# 자신부터 접속을 재 사용한다
net.ipv4.tcp_tw_reuse = 1

# RFC1337을 지킨다
net.ipv4.tcp_rfc1337 = 1

# 시스템이 동시에 유지하는 time-wait 소켓 최대 수
net.ipv4.tcp_max_tw_buckets = 360000

# 클라이언트에서 Sync 패킷은 버리지 않으면서 Syn cookies를 이용한 통신을 할 것인가의 설정
net.ipv4.tcp_syncookies = 1

# accept 큐가 넘친 경우 클라이언트로부터의 SYN 에 RST를 돌려준다
net.ipv4.tcp_abort_on_overflow = 0

# SYN flood 공격 대책
net.ipv4.tcp_syncookies = 1  
```
  
