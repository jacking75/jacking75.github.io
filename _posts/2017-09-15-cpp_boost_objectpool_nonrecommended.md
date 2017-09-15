---
layout: post
title: C++ - Boost의 object_pool 사용은 비 추천
published: true
categories: [C++]
tags: c++ boost object_pool
---
boost 라이브러리에는 메모리 풀 라이브러리로 object_pool 이라는 것이 있다.  
사용법은 아래와 같다.  
```C++
boost::object_pool<int> pool;
int* p = pool.construct(10);
pool.destroy(p);
```  
  
사용하면 안 되는 이유는 **사용한 오브젝트를 해제할 때 너무 느리기 때문**이다.  
[검증 코드](https://wandbox.org/permlink/su2qiqXBjldezpKl)  
  
느린 이유는 오브젝트의 pool 리스트를 쌍방향 리스트로 관리하고 있어서 삭제 시(메모리 해제 시) 리스트 전체를 검색하기 때문이다.  
이 리스트 검색인 O(N)으로 pool 에 있는 오브젝트가 많으면 많을 수록 느려진다(반대로 오브젝트 개수가 작으면 큰 문제가 되지 않는다).  
  
이 문제는 2009년 12월에 이미 이슈에 등록된 것인데 아직도 고치고 있지 않다고 한다.  
boost의 메모리 풀 쪽은 현재 유지 보수가 안 되고 있다고 한다.  
  
<br><br>  
  
출처: http://qiita.com/akinobufujii/items/02cb994c577d968283a1  
  