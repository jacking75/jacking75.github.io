---
layout: post
title: Rust - Vec 요소의 end 위치의 슬라이스를 함수의 인자로 넘기고 함수에서 len으로 크기 알아보기
published: true
categories: [Rust]
tags: rust Vec
---
[Permalink to the playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=298419e5ae06905f5058c3d78f29dac4  )  
  
```
fn main() {
    let xs: [u8; 5] = [1, 2, 3, 4, 5];
    test(&xs[5..]);
}

fn test(data: &[u8]) {
    println!("size: {}", data.len());
}
```
  
출력  
<pre>
size: 0
</pre>  
  