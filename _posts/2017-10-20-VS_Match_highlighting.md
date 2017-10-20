---
layout: post
title: Visual Studio - 일치한 부분 강조 표시(Match highlighting)
published: true
categories: [Visual Studio]
tags: vs vc vs2017
---
VS2017 에서는 C++은 미 지원.  
   
결과를 빨리 얻기 위해서 카멜 케이스 매칭(한 묶음으로 한 합성어의 각 대문자를 입력하여 보완) 등을 할 경우 아이템이 어느 부분에서 일치하는지 판단하기 힘들다.  
C#, VB, JavaScript, TypeScript의 IntelliSense에서는 아래 그림과 같이 일치한 곳을 굵은 글씨로 표시함으로써 항목이 일치하는 이유를 알게 되었다.  
일치 되는 곳을 강조 표시는 확장 포인트이며, 모든 언어에서 지원하도록 대응 언어는 향후 추가될 예정이다.  
  
![](/images/vs/vs_2017_0715_02.PNG)  
    
필터링이나 일치 부분 강조 표시 기능을 사용하고 싶지 않으면 사용 언어의 IntelliSense 설정으로 쉽게 무효화할 수 있다.
[Tools]-[Options]-[Text Editor] 에서 사용하는 언어를 선택한다.  
가령 [Tools]-[Options]-[Text Editor]-[C#]-[IntelliSense]  순서로 선택하면 C# 언어에서의 기능을 제어할 수 있다.    
  
출처: https://blogs.msdn.microsoft.com/visualstudio/2016/11/28/productivity-in-visual-studio-2017-rc/  