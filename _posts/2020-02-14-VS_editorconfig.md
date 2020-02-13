---
layout: post
title: Visual Studio - 개행 코드 설정 방법
published: true
categories: [Visual Studio]
tags: vs visualstudio vs2019 .editorconfig
---
.editorconfig 파일을 만들면 쉽게 할 수 있다.  
  
.editorconfig 파일  
```
[*]
end_of_line = lf
charset = utf-8-bom
```
  
end_of_line 에서 개행 코드를 LF 지정으로 한다.  
Visual Studio는 Bom 있는 UTF-8로 하는 것이 좋다  
  
이 .editorconfig 파일을 Visual Studio 프로젝트 안에 둔다.  


