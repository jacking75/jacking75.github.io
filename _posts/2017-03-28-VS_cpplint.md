---
layout: post
title: Visual Studio - cpplint 사용하기
published: true
categories: [Visual Studio]
tags: vs vc c++ vs2017 cpplint
---
### cpplint ?  
cpplint는 python으로 만든 C++ 소스 코드가 [Google C++Style Guide](https://google.github.io/styleguide/cppguide.html)를 지키고 있는지 검사하는 툴이다.  
  
<br>    
<br> 

### cpplint 설치
python 2.7에서 pip로 설치한다.  
  
<img src="resource\vc_cpplint01.PNG"> 
      
<br>    
<br> 


### Visual Studio에 외부 툴로 cpplint를 등록 
VS의 메뉴에서 [도구]-[외부 도구]를 선택한다.  
아래와 같이 입력한다.  
<img src="resource\vc_cpplint02.PNG"> 

VS의 메뉴에서 [도구]를 선택하면 아래와 같이 cpplint가 등록 되어 있다.  
<img src="resource\vc_cpplint03.PNG"> 
  
<br>    
<br> 
  

### cpplint로 c++ 코드 분석하기
VS의 메뉴에서 [도구]-[cpplint]를 선택하여 cpplint가 코드를 분석하도록 한다.  
그러면 아래처럼 분석 결과과 출력 창에 나온다.  
<img src="resource\vc_cpplint04.PNG"> 