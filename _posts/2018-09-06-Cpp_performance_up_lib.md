---
layout: post
title: C++ - 라이브러리를 사용한 프로그램 고속화
published: true
categories: [C++]
tags: c++ vc tcmalloc jemalloc nedmalloc
---
### tcmalloc, jemalloc, nedmalloc
- malloc 대체에 의한 고속화.
- 아래와 같이 실행함으로써 malloc이 대체되어 실행된다.
- LD_PRELOAD=/usr/lib/libtcmalloc.so ./exefile
   
상황에 따라 다르겠지만 tcmalloc이 제일 좋은 듯.
   
   
### google::dense_hash_map  
- tcmalloc에 이어 google 라이브러리가로 Hash Map을 사용할 때의 고속화이다.
- 아래 링크에 다양한 Hash Map을 이용했을 때의 벤치 마크 테스트 결과가 있지만 실행 시간에서 꽤 빠르다.  
  http://incise.org/hash-table-benchmarks.html  
   
생성자 호출 후에 empty_key를 설정할 필요가 있고, 무효한 키를 만들기 어려울 때는 사용하기 어렵다.  
   
   
### Boost::fast_pool_allocator
- Allocator를 메모리 풀을 이용하게 해서 고속화 하는 방법이다.  
  
```
std::vector<int, boost::fast_pool_allocator<int>> v;
``` 
   
     
### SIMD
- Intrinsic 명령이라는 사용 방법도 있지만 용도에 맞추어 SIMD 대응의 라이브러리를 사용하면 편리하다.
- [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page ) 행렬 연산 라이브러리
- [Boost.SIMD](https://github.com/jfalcou/nt2/tree/master/modules/boost/simd ) SIMD 명령 추상화
- [Libsimdpp](https://github.com/p12tic/libsimdpp )
- [Vc] (https://github.com/VcDevel/Vc )
   
<br>     
<br>  
   
출처: http://qiita.com/neka-nat@github/items/3cddc63123680a69dfeb
  