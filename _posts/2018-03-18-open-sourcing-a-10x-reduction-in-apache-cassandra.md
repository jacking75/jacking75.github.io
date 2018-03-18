---
layout: post
title: Cassandra의 스토리지 엔진을 바꾸어서  고속화한 Rocksandra를 Instagram이 오픈소스로 공개
published: true
categories: [Network]
tags: network dpdk
---
[원문](http://www.publickey1.jp/blog/18/cassandrarocksandrainstagramjava.html)  
        
Instagram은 NoSQL 데이터베이스 Cassandra에서 발생했던 Java의 가베지 컬렉션에 기인한 지연을 해소한 고성능 개량 버전 Cassandra를 오픈 소스로 공개했다고 발표했다.  
https://engineering.instagram.com/open-sourcing-a-10x-reduction-in-apache-cassandra-tail-latency-d64f86b43589  
    
스토리지 엔진인 key-value 스토어 "RocksDB"를 이용했다는 이유로 이 개량 버전 Cassandra를 "Rocksandra"라고 부르고 있다.  
  
Instagram에서는 Cassandra를 사용하고 있었는데 종종 레이턴시가 악화 된다는 문제를 안고 있었다.  
  
![](https://cdn-images-1.medium.com/max/800/1*ItBORNwCXce82ZNX6qf6Vg.png)     
위 그림에서 파란 색 선이 평균적인 레이턴시를 나타내고 오렌지 선은 99퍼센트(값 전체 중 좋은 쪽부터 세어서 100분의 99의 위치에 있는 값)의 레이턴시를 나타내고 있다. 즉, 전체 처리의 일부에서 종종 큰 지연이 발생하고 있다.  
  
조사 결과 이것이 Java의 가베지 컬렉션에서 기인한다는 것을 알아냈다.  

>>> 이 가베지 컬렉션의 오버 헤드는 99퍼센트의 레이튼 시에 큰 영향을 주고 있다. 만약 이 가베지 컬렉션에 의한 성능 저하의 비율을 낮추면 99퍼센트의 레이튼 시를 크게 줄일 수 있을 것이다.  
  
  
Java의 가베지 컬렉션 영향을 제거하기 위해 Instagram은 Cassandra의 스토리지 엔진을 대체하기로 했다.  
다만 처음부터 신규로 스토리지 엔진을 개발하면 리스크가 높아서 C++로 만들어진 오픈 소스 RocksDB를 베이스로 개발을 추진하기로 했다.  
  
RocksDB는 Google이 개발한 NoSQL 경량 라이브러리 "LevelDB"를 사용하여 Facebook가 개발한 key-value 데이터 스토어dl다.  
그러나 Cassandra의 스토리지 엔진을 교체에 몇 가지 과제가 있었다.  
첫 번째는, 원 Cassandra는 스토리지 엔진을 교체할 수 있는 설계가 되지 않다는 점. 그래서 동사는 스토리지 엔진을 Pluggable 하기 위해 API부터 만들기로 했다.  
>>> 대규모 리팩터링과 빠른 이테레이션 사이의 적절한 균형을 찾아 신규로 스토리지 엔진 API를 정의하기로 했다. 여기에는 가장 흔히 쓰이는 read/write나 스트리밍 인터페이스가 포함되고 있다. 이 덕분에 우리는 API 경유로 새로운 스토리지 엔진을 장착하고, Cassandra 내부의 관련된 코드를 경유해서 짜넣는 것이 가능했다.  
  
두 번째는, 스토리지 엔진이 되는 RocksDB가 단순한 key-value 데이터베이스, Cassandra가 풍부한 데이터 타입이나 테이블의 스키마를 지원하고 있다는 점이다. 이것을 RocksDB에 구현하기 위해 데이터 변환을 위한 인코딩과 디코딩 알고리즘을 이용했다.  
  
### 레이턴시 대폭 삭감에 성공
이러한 몇가지 과제를 극복하여 RocksDB를 Cassandra의 스토리지 엔진으로 구현한 "Rocksandra"의 벤치 마크가 아래이다.  
![](https://cdn-images-1.medium.com/max/800/1*Mpvc-jd61xmcrE4aEth4NA.png)   
파란색 마커가 Cassandra 3.0, 오렌지색  마커가 Rocksandra 이다.  
  
가장 왼쪽의 평균적인 레이턴시에서도 Cassandra가 2.267ms, Rocksandra가 1.386ms로 고속화를 이루어, P99에서도 11.864ms과 5.722ms로 절반 정도의 레이턴시.  
P 999에서는 263.21ms와 14.23ms로 압도적인 차이가 났다.  
  
Instagram에서는 앞으로 오픈 소스에서의 Rocksandra의 개발을 진행하여 세컨드리 인덱스나 리페어 라는 아직 구현되지 않은 Cassandra의 기능에 대한 추종 및 Cassandra에 대한 Pluggable 스토리지 엔진 아키텍처 구현에 대한 공헌을 해 나갈려고 한다.  

   
  