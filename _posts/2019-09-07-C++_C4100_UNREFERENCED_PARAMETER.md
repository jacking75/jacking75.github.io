---
layout: post
title: C++ - C4100 경고를 없애주는 UNREFERENCED_PARAMETER 매크로
published: true
categories: [C++]
tags: c++ UNREFERENCED_PARAMETER C4100 vc vc++
---
VC에서 경고 레벨 4로 컴파일 하면 자주 볼 수 있는 경고주 하나가 아래와 같은 경고이다.  
<pre>
warning C4100: 'argc' : unreferenced formal parameter
warning C4100: 'argc' : 참조되지 않은 형식 매개 변수입니다.
</pre>
  
  
선언은 했는데 사용하지 않았기 때문에 나온 것이다.  
이런 경우 해당 선언을 제거해도 되지만 장래에 사용될 수도 있거나, 구조적으로 제거를 할 수 없을 때가 있다.  
이럴땐 UNREFERENCED_PARAMETER 매크로를 사용하면 위의 경고를 피할 수 있다.  
  
``` 
void SomeFunction(int arg1)
{
    UNREFERENCED_PARAMETER(arg1);
}
```
  
