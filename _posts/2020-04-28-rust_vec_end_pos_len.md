---
layout: post
title: Rust - Vec 요소의 end 위치의 슬라이스를 함수의 인자로 넘기고 함수에서 len으로 크기 알아보기
published: true
categories: [Rust]
tags: rust Vec
---
[Permalink to the playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=298419e5ae06905f5058c3d78f29dac4  )  
  
https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&code=fn%20main()%20%7B%0A%20%20%20%20let%20xs%3A%20%5Bu8%3B%205%5D%20%3D%20%5B1%2C%202%2C%203%2C%204%2C%205%5D%3B%0A%20%20%20%20test(%26xs%5B5..%5D)%3B%0A%7D%0A%0Afn%20test(data%3A%20%26%5Bu8%5D)%20%7B%0A%20%20%20%20println!(%22size%3A%20%7B%7D%22%2C%20data.len())%3B%0A%7D