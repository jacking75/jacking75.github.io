---
layout: post
title: golang - 고루틴과 콜스택
published: true
categories: [Golang]
tags: go goroutine callstack
---
고루틴이 기동한 시점에서 함수의 콜스택이 분리된다.  
panic은 고루틴의 콜스택을 돌아간다.  
즉 defer & recover는 panic이 발생한 고루틴 내에서 사용해야 한다.  
  
  
![goroutine_callstack](../images/goroutine_callstack01.PNG)  
일바 함수 호출      

<br/>

![goroutine_callstack](../images/goroutine_callstack02.PNG)  
고루틴을 포함한 함수 호출  
    
