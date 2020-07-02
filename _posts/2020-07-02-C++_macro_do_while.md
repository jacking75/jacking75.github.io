---
layout: post
title: C++ - DO-WHILE에 의한 매크로 트랩핑
published: true
categories: [C++]
tags: c++ macro dowhile
---
[출처](http://cpp.aquariuscode.com/do-while-macro  )  
  
```
#ifdef _DEBUG
  #define DEBUG_LOG(exp, ...)  do {if (!(exp)){std::printf(__VA_ARGS__);}} while(0)
#else
  #define DEBUG_LOG(exp, ...)  do {(void)(exp);} while(0)
#endif

int main() {
    int x = 99;
    DEBUG_LOG(x > 100,"warning! x = %d", x); // x가 100 이하라면 경고를 표시
    return 0;
}
```  
  
<br>  
  
`#define DEBUG_LOG(exp,...)   if ((!exp)){std::printf(__VA_ARGS__)}`의 문제점  
1. 장래에 `;`(세미 콜론)을 강제할 수 없다  
2. 기술하는 장소에 의해서 의도하지 않는 행동이 된다  
3. 빌드 레벨에 의해서 기술할 수 있는 요소가 바뀐다  
  
  
## 장래에 `;`(세미 콜론)을 강제할 수 없다  
  
```
int main()
{
    int x=99;
    
    if(!(x>100) {
        std::printf("warning!x=%d", x);
    }
    
    return 0;
}
```  
  
```
int main() 
{
    int x = 99;
    DEBUG_LOG(x > 100,"warning! x = %d", x) //← 세미콜론이 없음
    return 0;
}
```
  
이것은 C++ 구문에 준하는 않고, 코드 전체의 일관성도 없다.  
그러나 이렇게 쓰는 것은 사용자 측이 아니라 정의한 측의 책임이다.  
  
  
  
## 기술하는 장소에 의해서 의도하지 않는 행동이 된다  
  
```
if (p)
    DEBUG_LOG(x > 100, "warning!");
else
    p = nullptr;
```  
코드는 아래처럼 된다.  
```
if (p)
    if (!(x > 100)) {
        std::printf("warning!");
    }
    else
        p = nullptr;

```  
매크로에서 do-while을 사용하면 아래처럼 된다.
```
if (p)
    do {
        if (!(x > 100)) {
            std::printf("warning");
        }
    } while(0);
else
   p = nullptr;
```  
  
  
## 빌드 레벨에 의해서 기술할 수 있는 요소가 바뀐다    
  
```
DEBUG_LOG(x > 100, "warning!")
else {
    //// 어떤 처리...
}
```  
if문 매크로의 경우는 이런 서식이 C++적으로 허용된다.
그러나 디버깅 빌드 시는 문제 없는데 정작 릴리스로 하면 컴파일 오류로 되어 버린다.  
  
이런 것이 비 디버깅 시 DEBUG_LOG의 정의에도 해당된다.  예를 들면 `(void)0)`을 안 쓰고, 전개 결과를 그저 빈 칸으로 만들어 버리면 비 디버그 빌드 시에는 아무 데나 기술할 수 있는 매크로가 버린다.  
```
#else
  // 릴리즈 시에는 어떤 전개로 하지 않는다
  #define DEBUG_LOG(exp, ...)
#endif

int main(DEBUG_LOG(1)) {
    return DEBUG_LOG(0) 0;
}
```
  
매크로는 C++ 문법을 무시하고 아무 데나 기술할 수 있다.  
아무데나 기술하려는 뚜렷한 이유가 없는 경우 이러한 기술은 하지 못해야 하고, 원래 디버그 빌드 시와 기술 가능 부분의 규정이 바뀌어 버린다.  
  
빌드 레벨에 의해서 기술 가능 부분의 규칙이 변하는 것은 단순히 이해하기 힘든 것 이상으로 빌드 레벨을 바꾼 타이밍에서 처음으로 코딩의 실수를 깨닫게 되므로 효율도 나쁘다.  
  
  
## 정리
전개될 때 의도하지 않는 행동이 되는 것을 막기 위해서 매크로를 do-while(0)로 묶는 것은 좋은 사례.  
빌드 레벨이 변해도 매크로를 펼치는 곳에는 일관성을 가져야 한다.  
  