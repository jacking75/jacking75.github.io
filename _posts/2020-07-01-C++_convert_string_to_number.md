---
layout: post
title: C++ - 문자열을 수치로 변환하는 방법
published: true
categories: [C++]
tags: c++ atoi strtol sscanf stoi istringstream boost::lexical_cast boost::spirit::qi
---
## atoi
  
```  
#include <cstdlib>
#include <iostream>
#include <string>
#include <typeinfo>

int main()
{
    const std::string str("123");
    auto num = std::atoi(str.c_str());
    std::cout << typeid(num).name() << " : " << num << std::endl;
}
```  
  
  
## strtol
  
```
#include <cstdlib>
#include <iostream>
#include <string>
#include <typeinfo>

int main()
{
    const std::string str("123");
    char* e = nullptr;
    auto num = std::strtol(str.c_str(), &e, 10);
    std::cout << typeid(num).name() << " : " << num << std::endl;
}
```
  
  
## sscanf
  
```
#include <cstdio>
#include <iostream>
#include <string>
#include <typeinfo>

int main()
{
    const std::string str("123");
    int num = 0;
    sscanf(str.c_str(), "%d", &num);
    std::cout << typeid(num).name() << " : " << num << std::endl;
}
```
  
  
## stoi
  
```
#include <iostream>
#include <string>
#include <typeinfo>

int main()
{
    const std::string str("123");
    int num = std::stoi(str);
    std::cout << typeid(num).name() << " : " << num << std::endl;
}
```
  
  
## istringstream
  
```
#include <iostream>
#include <sstream>
#include <string>
#include <typeinfo>

int main()
{
    const std::string str("123");
    std::istringstream iss(str);
    int num = 0;
    iss >> num;
    std::cout << typeid(num).name() << " : " << num << std::endl;
}
```
  
  
## boost::lexical_cast
  
```
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <typeinfo>

int main()
{
    const std::string str("123");
    auto num = boost::lexical_cast<int>(str);
    std::cout << typeid(num).name() << " : " << num << std::endl;
}
```
  
  
## boost::spirit::qi
  
```  
#include <boost/spirit/include/qi.hpp>
#include <iostream>
#include <string>
#include <typeinfo>

int main()
{
    const std::string str("123");
    int num = 0;
    boost::spirit::qi::parse(str.begin(), str.end(), boost::spirit::qi::int_, num);
    std::cout << typeid(num).name() << " : " << num << std::endl;
}
```
  