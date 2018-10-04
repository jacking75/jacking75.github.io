---
layout: post
title: Linux에서의 최대 스레드 수
published: true
categories: [OS]
tags: OS Linux thread
--- 
Linux에서 스레드의 최대 수를 변경하고 싶다면 커널의 파라메터를 조정하면 된다.  
kernel.threads-max와 kernel.pid_max, vm.max_map_count 수늘 늘리면 된다.  

예)  
```
sysctl -w kernel.threads-max=600000
sysctl -w kernel.pid_max=600000
sysctl -w vm.max_map_count=600000
```  
  