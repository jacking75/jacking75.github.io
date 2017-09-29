---
layout: post
title: Visual Studio - C++ 솔루션 로딩과 빌드 고속화
published: true
categories: [Visual Studio]
tags: vs vc vs2017
---
[MSDN에 있는 글](https://blogs.msdn.microsoft.com/visualstudio/2016/10/13/faster-c-solution-load-and-build-performance-with-visual-studio-15/) 의 번역 입니다.  
  
## C++ 솔루션 로딩 고속화
C++ 프로젝트를 향해서 "신속한 프로젝트 로딩"이라는 시험적인 기능이 도입되었다.  
C++ 프로젝트를 처음 열었을 때의 로딩 시간을 단축하는 것으로 2번째 이후에는 더 짧아진다.  
이 시험 기능을 사용하는 경우는 아래와 같이 [Tools]-[Options] 순서로 선택한 후 [Enable Faster Project Load]를 [True]로 설정하자.  
  
![](/images/vs/vs_2017_0715_03.PNG)  
    

다음의 간단한 데모에서는 1968개의 프로젝트를 포함한 대규모 Chromium Visual Studio 솔루션을 이용하여 로딩 시간이 단축되는 것을 증명하고 있다.  
자세한 것은 [C++ 솔루션 로딩 고속화에 관한 블로그 기사(영어)](https://blogs.msdn.microsoft.com/vcblog/2016/10/05/faster-c-solution-load-with-vs-15/)를 참조하자.  
  
이와 별도로 Visual Studio에서는 솔루션의 로딩을 고속화하는 "라이트 웨이트 솔루션 로드" 기능 시험도 하고 있다.  
이 2개의 기능에는 전혀 다른 방법이 사용되고 있다. 자세한 내용은 [이 기사](https://aka.ms/vs/15/preview/vs_enterprise)를 참조한다.  
  
라이트 웨이트 솔루션 로드 기능은 모든 프로젝트를 내놓고 읽는 것이 아니라 솔루션 익스플로러에서 프로젝트가 명시적으로 확장 되었을 때 해당 프로젝트만 읽는 것이다.   C++ 팀은 신속한 프로젝트 로딩 기능에 집중적으로 노력하고 있어서 현 시점에서는 라이트 웨이트 솔루션 로드 지원은 최소한에 머무르고 있다.  
RC판 Visual Studio 15에선 라이트 웨이트 솔루션 로드와 신속한 프로젝트 로딩 기능 모두 지원할 예정인데 이 2가지를 조합하면 뛰어난 경험이 실현된다.  
  
  
## /debug:fastlink로 빌드 사이클 단축
개발자 빌드에서 /debug:fastlink 경험과 통합에 의한 링크가 고속화 되어 빌드 시간이 단축되었다.  
링커의 개량으로 애플리케이션의 빌드 시간이 2분의 1에서 4분의 1로 단축된다.  
  
다음 그림에서는 C++의 주요 소스에 대한 링크 시간이 /debug:fastlink에 의해 어느 정도 단축됐는지를 나타내고 있다.  
/debug:fastlink의 상세나 Visual Studio와의 통합에 대해서는 최근 공개된 [C++의 빌드 사이클 단축에 관한 블로그 기사(영어)](https://blogs.msdn.microsoft.com/vcblog/2016/10/05/faster-c-build-cycle-in-vs-15-with-debugfastlink/)를 참조하자.  
  
![](/images/vs/vs_2017_0715_02.PNG)  
    
	
## 메모리 부족에 의한 디버깅 중의 크래시 감소
VS"15"Preview 5(VS 2017 출시 전의 이름)에서는 심볼 데이터의 사전 취득에 의한 향상된 퍼포먼스를 유지하면서 심볼 정보가 사용하는 메모리의 양을 줄였다.  
이번 버전에서는 변수나 식 평가 및 표시에 관련 모듈만 사전 취득 되도록 하였다.  
그 결과 VS 2015에서는 메모리 부족으로 깨졌던 Unreal Engine 프로세스의 디버깅을 VS"15"Preview 5에서 했을 때 가상 메모리 사용량은 3GB이하, 프라이베이트 워킹 셋 사이즈는 1.8GB로 묶이며 크래시가 발생하지 않았다.  
분명히 개선되지만 만족스러운 결과는 아니다.  
마이크로 소프트는 향후 VS"15" 개발에서도 네이티브 디버깅 작업 시나리오에서 메모리 사용량 삭감을 더 진행할 예정이다.  
  