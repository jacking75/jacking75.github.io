---
layout: post
title: golang - 코드 정적 분석 툴 go vet
published: true
categories: [Golang]
tags: go golang vet
---
아래 글은 golang을 공부할 목적으로 웹에서 본 글들을 정리한 것이다.  
  
## go vet
- go의 표준 패키지에 포함된 코드 정적 분석 툴이다.
- 공식 문서는 https://golang.org/cmd/vet/ 
  
  
## go vet 사용 방법 ・flag 등

```
go vet package/path/name # package 단위
go tool vet /path/to/*.go # file 단위
go tool vet /path/to/directory # directory 단위
```  
  
go doc cmd/vet 에서 document도 보인다.    
default로 -all 이 유효하므로 일부 의도적으로 disabled한 물건이 숨겨져 있는 것은 특히 지정하지 않아도 감지 가능하다.  
  
  
### go vet 이 감지 해주는 것들
- Unreachable code
- Shadowed variables
- bad syntax for struct tag value
- no args in Error Call
- Mutex Lock
- UnSafePointer
  
로직 상 절대로 도달하지 않는 코드가 있다고 하거나, 인수에 명시적으로 nil을 넘겨라고 말하는 세세함에서 Json 이나 Xorm 등 구조체의 프로퍼티를 태그를 붙인 경우에 가르쳐주거나, 컴파일에서 몰랐지만 실행 때 문제가 되는 것도 있다.    
    