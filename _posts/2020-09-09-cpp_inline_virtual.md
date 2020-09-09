---
layout: post
title: C++ - 가상함수의 inline화에 대해서
published: true
categories: [C++]
tags: c++ virtual inline
---
출처: Web의 어딘가...  
  
혹시 가상함수는 inline 되지 않는다고 생각하는 사람이 있을  것이다.   
경우에 따라서 되기도 하고 안 되기도 한다.  
되는 경우는 컴파일 시점에서 어떤 클래스의 가상함수를 사용하는지 알 수 있느냐 이다.    
  
아래 코드의 경우 컴파일 시점에서 Delived 클래스의 Function 멤버를 사용한다는 것을 알 수 있기 때문에 가상함수인 Function은 Inline화 할 수 있다.  
```
struct Base
{
    virtual void Function(void) const = 0:
};

struct Delived
{
    virtual void Function(void) const
    {
         std::cout << "Hello" << std::endl;
    }
};

int main(int, char**)
{
    Delived  delived;
    delived.Function();
    return 0;
}
```  
  
  
그럼 아래 코드의 Function 멤버는 Inline화 할 수 있을까?   
물론 된다.  
이유는 이것도 컴파일 시점에서 어떤 클래스의 Function을 사용하는지 알 수 있기 때문이다.  
```
struct Base
{
    virtual void Function(void) const = 0:
};

void Call( Base &base )
{
    base.Function();
}

struct Delived:  public Base
{
    virtual void Function(void) const
    {
         std::cout << "Hello" << std::endl;
    }
};

int main(int, char**)
{
    Delived   delived;
    Call( delived );
    return 0;
}
```
  
  
아래의 코드는 Function 멤버를 Inline화 할 수 없다.  
```
struct Base
{
    virtual void Function(void) const = 0:
};

struct Delived:  public Base
{
    virtual void Function(void) const
    {
         std::cout << "Hello" << std::endl;
    }
};

struct Null  : public Base
{
    virtual void Function(void) const{}
};

int main( int argc, char** )
{
    Delived   delived;
    Null        null;

    Base  *base = 1 < argc ? static_cast<Base*>( &delived ) : static_cast<Base*>( &null );
    base->Function();

    return 0;
}
```
  

