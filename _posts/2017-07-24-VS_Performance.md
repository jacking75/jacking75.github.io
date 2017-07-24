---
layout: post
title: Visual Studio - VS2017의 성능 강화
published: true
categories: [Visual Studio]
tags: vs vc vs2017
---
  
- 프로젝트의 읽기를 경량화 하고, 로딩 시간을 단축: 100 종류등 많은 프로젝트가 포함 되어 있는 솔루션에서는 모든 파일이나 프로젝트가 동시에 사용되지 않는다.  
VS 2017에서는 모든 프로젝트를 읽기 전까지 기다리지 않고 편집과 디버깅을 시작할 수 있다.   
  
- 확장 기능의 로딩을 온디멘드화 하여 시작 시간을 단축: 이 아이디어는 아주 심플하여 단순히 확장 기능의 로딩을 VS 시작 시가 아닌 필요할 때 실행한다는 것이다.  
우선 Python과 Xamarin의 확장 기능의 로딩을 온디멘드화 했다. 이 후에는 Visual Studio에 동봉하는 마이크로 소프트 제품과 서드 파티 제품의 모든 기능도 이 모델로 이행한다. 시작과 솔루션의 로딩, 키 입력의 퍼포먼스에 영향을 주는 확장 기능에 관한 정보는 [Help]-[Manage Visual Studio Performance]의 순서로 선택하여 확인하자.  
또 확장 기능 개발자들에게 온디멘드화에 관한 가이드 라인을 공개할 예정이다.  

- 서브 시스템을 VS의 메인 프로세스에서 분리: Git Source Control 등 메모리 소비량이 많은 태스크나 JavaScript나 TypeScript의 언어 서비스의 프로세스를 분리했다.  
32비트 처리에서는 메모리가 4GB로 제한되지만, 프로세스 분리에 의해 메인 프로세스가 이 제한을 회피할 수 있어 Visual Studio의 메인 프로세스에서 실행되는 코드의 속도 저하나 Visual Studio의 응답 속도 저하나 크러시가 방지된다. 향후 릴리스에서는 더 컴포넌트의 프로세스 분리를 진행할 예정이다.  

- C++ 프로젝트 로딩, 코드 편집, 디버깅을 고속화: C++ 프로젝트의 로딩이 고속화 되었다. 이 기능을 활성화하려면 [Tools]-[Options]-[Text Editor]-[C/C++]-[Experimental] 순서로 선택하고 [Enable Faster Project Load]를 on 한다. 또 링커나 PDB 읽기 라이브러리가 개량되면서 차분 빌드가 만들어지도록 하였다. 게다가 디버거의 시작이 고속화되어 디버깅 작업 중의 메모리 소비량이 대폭 감소했다.  
  
- Git 사용 중이나 XAML 코드 디버깅/편집 중 속도 향상: libgit2에서 git.exe로 전환함으로써 소스 관리 조작이 고속화 되었다. 또한 초기화 비용과 IntelliTrace 및 [Diagnostic Tools] 윈도 관련 기타 비용이 최적화되어, XAML 파일 편집 시의 지연 발생이 잘 일어나지 않는다.  
  
  
  
참고: https://blogs.msdn.microsoft.com/visualstudio/2016/10/05/announcing-visual-studio-15-preview-5