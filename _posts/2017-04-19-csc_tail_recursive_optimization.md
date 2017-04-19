---
layout: post
title: C# - 꼬리 재귀 최적화
published: true
categories: [.NET]
tags: c# .net 재귀 꼬리_재귀_최적화
---

```
// 꼬리 재귀가 아니다
long SumNormal(long n)
{
    if (n <= 0) return 0;
    return SumNormal(n - 1) + n;
}

// 꼬리 재귀
long SumTailCall(long n)
{
    return SumTailCallAcc(n, 0);
}

long SumTailCallAcc(long n, long acc)
{
    if (n <= 0) return acc;
    return SumTailCallAcc(n - 1, acc + n);
}
```
  
위 코드를 (최적화 옵션으로)컴파일하면 SumNormal은 n = 100000 정도에서 스택오버플로우 하지만 SumTailCall는 스택오버플로우 하지 않는다.  
그리고 최적화 옵션을 주지 않으면 둘 다 스택오버플로우 한다.  
  
  
<br>
<br>  

출처: http://qiita.com/brln/items/05ea1d9288fab4934612