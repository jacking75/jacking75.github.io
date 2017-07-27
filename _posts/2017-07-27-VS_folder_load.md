---
layout: post
title: Visual Studio - 폴더(디렉토리) 선택으로 열기
published: true
categories: [Visual Studio]
tags: vs vc c++ vs2017
---
VS 2017 이전에는 VS 솔루션 파일이 소스 파일은 VS에서 사용할 수 없었다.  
VS에서 사용하려면 새로 솔루션을 만든 후 수동으로 소스 파일들을 추가해야 했다.  
그러나 VS 2017의 새로운 기능으로 폴더 선택 열기만으로 솔루션 파일이 없는 소스 파일들을 사용할 수 있다.    
  
VS2017를 설치 후 탐색기에서 특정 폴더를 선택 후 마우스 오른쪽 클릭을 하면 아래와 같은 메뉴가 나온다.  
 
![](/images/vs_0005.PNG)  
   
메뉴 선택을 하면 아래와 같이 솔루션을 만들면서 자동으로 파일을 솔루션에 추가해 준다.     
![](/images/vs_0006.PNG)     
    
<br>  
      
VS 2017에서는 CMake 파일도 지원하므로 폴더 열기를 하는 경우 CMake 파일이 있으면 자동으로 빌드까지 해준다.  
  
![](/images/vs_0007.PNG)  
![](/images/vs_0008.PNG)  
  
