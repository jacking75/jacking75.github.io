---
layout: post
title: Google에서 3DCG를 위한 압축 라이브러리 「Draco」를 GitHub에 공개 
published: true
categories: [번역]
tags: 번역 draco
---
https://github.com/google/draco

Chrome Media팀이 개발한 압축 라이브러리로 ZIP 형식보다 효율적으로 데이터 압축을 실행할 수 있는 오픈 소스이다.  
압축한 Mesh 파일의 크기 100MB 일 때 ZIP으로는 30MB 이지만, Draco에서는 10MB 이하가 되어 훨씬 압축 효과가 있다고 한다. 
  
Draco는 메시와 점군 데이터를 압축하기 위해서 사용 가능하며, 압축 포인트, 접속 정보, 텍스쳐 좌표, 색깔 정보, 법선 및 지오 메트리에 관련된 속성도 지원 하고 있다.  
엔코더는 C++로 구현되어 있으며, 디코더는 C++ 및 JavaScript로 구현되어 있다.  
  
Draco를 사용하면 손상을 억제하는 앱 다운로드 속도를 향상시키거나 브라우저의 3D그래픽을 보다 고속으로 읽어서 VR/AR에서도 대역을 대폭 줄이고, 빠르게 렌더링 하여 보기를 향상시킬 수 있다.  
  
   
<br>
<br>  

출처: http://shiropen.com/2017/01/17/22639