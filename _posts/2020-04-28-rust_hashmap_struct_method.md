---
layout: post
title: Rust - HashMap - 구조체의 메소드를 value로 사용하기
published: true
categories: [Rust]
tags: rust HashMap
---
아래 코드는 페이스북의 한국 러스트 사용자 그룹에서 내가 올린 질문에 김지현님이 올린 답글이다 
[Permalink to the playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=30b7e400a432a77a850403b60473a727  )  
  
```
use std::collections::HashMap;

pub struct MyType { /* ... */ }

impl MyType {
    fn new() -> Self {
        MyType { /* ... */ }
    }
    
    fn foo(&mut self) { /* ... */ }
    fn bar(&mut self) { /* ... */ }
}

fn main() {
    let mut a = MyType::new();
    let mut b = MyType::new();
    
    {
        // map 안에 a b에 대한 포인터가 들어있기때문에,
        // 반드시 map은 a b보다 짧게 살아야함
        let mut map = HashMap::<_, Box<dyn FnMut()>>::new();
        map.insert(0, Box::new(|| a.foo()));
        map.insert(1, Box::new(|| b.bar()));
    }
}
```
  
  