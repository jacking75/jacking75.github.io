---
layout: post
title: golang - exec.LookPath 커맨드 명령어가 실행 가능한지 조사
published: true
categories: [Golang]
tags: go exec.LookPath
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
os/exec.LookPath를 사용하면 커맨드가 실행 가능한지 조사 할 수 있다.  
$PATH 를 고려하여 찾아주지만 슬래쉬가 들어가 있으면 $PATH를 고려하지 않는다.  

```
package main

import (
    "log"
    "os/exec"
)

func main() {
    tests := []string{
        "hoge",
        "/bin/hoge",
        "./hoge",
    }

    for _, test := range(tests) {
        if _, err := exec.LookPath(test); err != nil {
            log.Print(err)
        }
    }
}
```  
실행 결과  
<pre>
2013/12/10 19:18:34 exec: "hoge": executable file not found in $PATH
2013/12/10 19:18:34 exec: "/bin/hoge": stat /bin/hoge: no such file or directory
2013/12/10 19:18:34 exec: "./hoge": stat ./hoge: no such file or directory
</pre>  
  

