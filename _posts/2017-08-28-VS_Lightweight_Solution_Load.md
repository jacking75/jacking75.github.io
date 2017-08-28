---
layout: post
title: Visual Studio - Lightweight Solution Load
published: true
categories: [Visual Studio]
tags: vs vc vs2017
---
[Lightweight Solution Load](한글로는 모든 솔루션에 대해 경량 솔류션 로드)를 활성화 하면 프로젝트 지연 읽기를 할 수 있다.  
(프로젝트 로딩 시 전체 솔루션이 로딩 다 되지 않아도 활성화 되면서 순차적으로 솔루션을 로딩한다. 즉 모든 솔루션이 로딩 될 때까지 대기하지 않을 수 있다)
이 기능은 솔루션에는 수 백개의 프로젝트가 포함 되지만 작업에 사용하는 프로젝트는 그 중 극소수인 경우에 특히 편리하다.  
  
[Lightweight Solution Load]를 활성화하는 경우는 [Tools]-[Options]-[Projects and Solutions]-[Lightweight solution load for all solutions]의 순서로 선택한다.  

  
