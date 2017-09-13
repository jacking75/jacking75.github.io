---
layout: post
title: Visual Studio - 업데이트 등에서 에러가 발생했을 때의 대처 방법
published: true
categories: [Visual Studio]
tags: vs vs2017
---
1. Open an elevated command prompt.
2. Run:
```
%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\resources\app\layout\InstallCleanup.exe -i
```    
  
위 조작으로 설치를 위해 다운로드된 것이나 사용한 것이 지워지고, 리셋 된다.  
    
%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\resources\app\layout\InstallCleanup.exe 를 찾을 수 없는 경우는 https://www.visualstudio.com/downloads
/  보다 최신 인스톨러를 다운로드하du 실행해 본다.   
 
   
**이것도 안 되면 깔끔하게 삭제한다.**  
  
<br>  
   
[출처](https://blogs.msdn.microsoft.com/heaths/2017/09/01/cleaning-up-corrupt-visual-studio-instances/)  

