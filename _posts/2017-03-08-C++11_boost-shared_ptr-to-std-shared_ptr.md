---
layout: post
title: C++11 - boost::shared_ptr 에서 std::shared_ptr로 변환 하는 가장 간단한 방법
published: true
categories: [C++]
tags: c++ c++11 boost shared_ptr
---
    
```
#include <iostream>

// std::shared_ptr
#include <memory>
// boost::shared_ptr
#include <boost/shared_ptr.hpp>

auto main() -> int
{
    boost::shared_ptr< int > b;
    
    {
        b = boost::shared_ptr<int>( new int( 123 ) );
        std::cout << b.use_count() << std::endl;
        
        std::shared_ptr< int > s = std::shared_ptr<int>( b.get(), [b] ( ... ) mutable { } );        
        std::cout << b.use_count() << std::endl;
    }
    
    std::cout << b.use_count() << std::endl;
}
``` 
  
<br>  
<br>  

출처: http://qiita.com/usagi/items/3563ddb01e4eb342485e

