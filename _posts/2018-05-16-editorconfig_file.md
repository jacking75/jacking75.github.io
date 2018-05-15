---
layout: post
title: editorconfig 파일
published: true
categories: [etc]
tags: github vs visualstudio
---
이것으로 프로젝트 디렉토리에  .editorconfig 파일을 준비해 두면 설정이 적용된다.  
(Visual Studio, Atom 등 유명 에디터에서 지원하고 있다)  
즉 git의 관리 대상으로 하면 프로젝트에서 통일 시킬 수 있다.  
  
.editorconfig 파일의 예  
```
# EditorConfig helps developers define and maintain consistent
# coding styles between different editors and IDEs
# editorconfig.org

root = true


[*]

# Change these settings to your own preference
indent_style = space
indent_size = 2

# We recommend you to keep these unchanged
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false
```  
  
### 설정 파일 쓰는 법
설정 파일은 위처럼 프로퍼티와 값의 리스트이다.  
어떤 프로퍼티가 있고 어떤 값을 사용하는 정리했다.  
- indent_style  
  하드 탭인지 소프트 탭인지. tab 이나 space를 지정할 수 있다.     
- indent_size  
  인덴트를 몇 개의 스페이스로 할지. 지정하는 것은 정수이다.   
- tab_width  
  탭의 폭인다. 생략하면 indent_size의 수치로 적용된다.    
- end_of_line  
  개행 코드의 종류가 cr 이나 crlf 를 지정한다.    
- charset  
  문자 코드. utf-8 지정을 추천.   
- trim_trailing_whitespace  
  행 끝의 공뱅을 삭제할지 어떨지. true 나 false를 지정한다.    
- insert_final_newline  
  최종 행에 빈 행을 넣을지 어떨지. true 나 false.    
- root  
  이것을 true로 하지 않으면 루트 디렉토리까지 찾아간다.    
  root 기능 덕택으로 .editorconfig가 같은 계층에 존재하지 않는 경우에 루트에 존재하는 .editorconfig 파일을 참조해 준다.    
  root = true 는 꼭 최초에 써둔다고 생각하자.    
   
   
### 파일 이름 지정 방법
특히 어려운 것은 없지만 공식 페이지에 있는 것을 남긴다.  
  
| 셀렉터   | 효과                   |
|------------|------------------------|
| *          | "/"을 제외한 임의의 문자열  |
| **         | 임의의 문자열           |
| ?          | 임의의 한 문자           |
| [name]     | name에 일치하는 것     |
| [!name]    | name에 일치하지 않는 것   |
| {s1,s2,s3} | s1,s2,s3 에 일치하는 각각 |
  
특수 문자에서 이스케이프는 백슬래쉬이다.  
주석은 #슬래쉬나 ;세미콜론 이다.  
  
    
### 샘플
```
root = true

[*]
indent_style = space
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
indent_size = 4

[*.md]
trim_trailing_whitespace = false

[package.json,*.less,*.coffee]
indent_size = 2
```  
  
### 참고
[EditorConfig](http://editorconfig.org/)  
  
<br>  
<br>     
  
[원본](https://qiita.com/naru0504/items/82f09881abaf3f4dc171)
