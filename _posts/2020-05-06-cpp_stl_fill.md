---
layout: post
title: C++ - C++로 다차원 배열 초기화 방법. std::fill()의 편리한 사용법
published: true
categories: [C++]
tags: c++ stl fill array
---
std::fill  
```
// n 차원 배열의 초기화. 제 2인수의 형의 사이즈마다 초기화 해 둔다
template<typename A, size_t N, typename T>
void Fill(A (&array)[N], const T &val){
    std::fill( (T*)array, (T*)(array+N), val );
}
```
  
예제 코드  
```
int main(){
    int a[10];
    Fill( a, 12 );      // 배열 a의 내용이 모두 12로 된다

    int b[10][20];
    Fill( b, 100 );     // 2차원 이라도 쓰는 방법은 같다

    int c[10][20][30];
    Fill( c, 100 );     // 복수 차원에서 사용 가능

    long long d[20][20];
    Fill( d, (long long)100 );    // 제２인수의 형을 배열 요소의 형에 맞추지 않으면 안 된다

    int N = 10;
    int e[N][2];
    //Fill( e, 100 );    // 에러. 동적으로 확보한 배열에는 사용할 수 없다

    return 0;
}

```  
  
<br>  
  
std::fill()만 사용할 때는 다음과 같이 초기화 할 수도 있다.  
```
int a[10][20];
std::fill( a[0], a[10], 100 );

int b[10][20][30];
std::fill( b[0][0], b[10][0], 100 );

// 앞의 Fill()이 하는 것
int array[10][20][30];
int val = 100;
std:fill( (int*)array, (int*)(array+10), val );

```  
  