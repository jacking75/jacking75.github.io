---
layout: post
title: C++ - delete 호출과 구현
published: true
categories: [C++]
tags: c++ delete
---
[출처](http://cpp.aquariuscode.com/delete_fact  )  
  
```
#define SAFE_DELETE(p)if(p){delete p;}
```  
  
옛날, 마이크로 소프트가 DirectX의 샘플에서 이런 매크로를 정의한 바 있다.  
아마도 많은 사람이 그 샘플의 대로 익힌 결과, 특히 생각 없이 구현하는 경우도 적지 않을것이다.  
  
포인터가 null이 아님을 판정한 뒤 delete하는 처리이다.  
  
delete 대상 포인터가 null로 드러날 경우 어떻게할까?  
미 정의 동작일까?  
접근 예외로 행업 하거나, 뉴저지 주에 사는 이모에게 허락 없이 메일을 보내는 동작을 일으키겠죠?  
답은 "아무 일도 일어나지 않는다"  
  
delete에 null 포인터를 줘도 안전하다고 규격에도 명기되어 있다.  
  
  
아는 사람에게는 당연하지만 의외로 모르고 C++를 사용하는 사람도 많을 것이다.  
null 인가 어떤지를 걱정하지 말고 포인터를 delete 할 수 있는 것 대신에 이번에는 주의해야 하는 것이 생겼다.  
스스로 delete 연산자를 정의하는 경우이다.    
```
//global delete를 사용자 정의 delete로 치환
void operator delete(
    void* p
) {
      if(p){
         user_defined_free(p); //사용자 정의 메모리 해제 처리
    }
}
```
규격에 준하기 위해서는 null을 허용하도록 구현하지 않으면 안 된다.  
  
  
## 정리
1. null 포인터를 delete에 건네줘도 문제 없다  
2. 오리지널 delete 연산자를 정의하는 경우는 null 포인터에 대응해야 한다  
  