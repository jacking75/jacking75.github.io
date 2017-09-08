---
layout: post
title: MySQL의 3가지 lock - Record lock, Gap lock, Next-key lock
published: true
categories: [DB]
tags: mysql lock
---
  
- Record lock(레코드 락): 단일 인덱스 레코드의 락.  
- Gap lock(갭 락): 인덱스 레코드 사이의 갭의 락, 선두 인덱스 레코드의 앞이나 말미 인덱스 레코드의 뒤의 갭의 락
- Next-key lock(넥스트 키 락):레코드 락과 이 레코드 직전의 갭 락의 조합  
  
InnoDB의 default 분리 레벨(REPEATABLE-READ)에서는 락 단위는 넥스트 키 락이 기본으로 어느 인덱스에 값 10,11,13,20 이 포함됐을 때 넥스트 키 락 구간은 아래와 같은 느낌으로 기본적으로 이 구간 단위로 잠근다.  
<pre>
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
</pre>  
  
<br>  
    
[참고](http://blog.kamipo.net/entry/2013/12/03/235900)  