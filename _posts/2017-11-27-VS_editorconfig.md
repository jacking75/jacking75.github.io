---
layout: post
title: Visual Studio - editorconfig 사용하기
published: true
categories: [Visual Studio]
tags: vs vc vs2017 editorconfig
---
VS 2017에서는 프로젝트 마다 혹은 프로젝트의 특정 폴더마다 에디터 설정을 바꾸기 원한다면 .editorconfig 파일을 사용한다.  
VS 2015에서는 이와 동일한 기능을 사용하려면 플러그인을 사용해야 했으나 2017에서는 기본 기능으로 들어왔다.  
  
.editorconfig  
```
root = true
  
[*]
indent_style = space
indent_size = 20
insert_final_newline = true
```
  
![](/images/vs/vs_2017_0803_01.PNG)  
  
위 파일을 VS 프로젝트에 넣은 후 프로젝트를 로드하면 이 파일이 있는 디렉토리의 에티터 설정은 위에서 설정한 것이 우선적으로 적용된다.
default로 C++의 경우 Tab 키를 누르면  tab이 적용되고 tab 사이즈는 4이다. 
그러나 위 파일이 있는 곳의 코드 파일은 Tab 키를 누르면 space가 적ㅇ요되고 사이즈는 20이 된다(즉 Tab키를 누를 때마다 빈공백이 20개 들어간다).

.editorconfig의 [] 부분은 그룹을 뜻하는데 [*]는 모든 파일에 적용되는 것을 뜻하고, [*.cs]는 C# 코드 파일에만 적용하고 싶을 때 사용하면 된다.   
  
예) 출처: http://srz-zumix.blogspot.kr/2017/06/visual-studio-2017-editorconfig.html    
```
root = true
  
[*]
indent_style = space
indent_size = 4
insert_final_newline = true

[Makefile]
indent_style = tab

[*.{cpp,hpp,ipp}}
charset = utf-8-bom

[*.py]
charset = utf-8

[*.{yml,html}]
indent_size = 2
```  

<br>  
  
설정 항목은 [여기](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties)를 참고한다.  
좀 더 자세한 설명은 [여기](https://docs.microsoft.com/ko-kr/visualstudio/ide/create-portable-custom-editor-options)를 참고한다.  
