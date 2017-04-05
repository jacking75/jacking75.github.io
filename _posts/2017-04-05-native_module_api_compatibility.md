---
layout: post
title: 네이티브 모듈의 ABI 호환성 문제 대처
published: true
categories: [Node.js]
tags: Node.js 번역 abi native
---
2017년 3월 13일(미국)에 열렸던 'Node.js VM Summit' 에서 Node의 네이티브 모듈의 ABI 호환성 문제에 대해서 회의를 하였다.  
[회의 동영상](https://www.youtube.com/watch?v=ZRVjWhBEwB0)  

### Node의 네이티브 모듈 문제
Node의 네이티브 모듈은 C/C++로 만들었고 V8과 NAN API(Application Programming Interface)에 직접 의존하고 있다.  
이들 모듈은 Node.js의 메이저 업데이트마다 ABI(Application Binary Interface)와 API의 변경에 의해 갱신이나 재 컴파일을 필요하게 될 가능성이 있다.  
  
이것은 네이티브 모듈 개발자에게 관리 부담이 늘어난다. 또 모듈 이용자에게 있어서도 실제 환경에서 사용하고 있는 Node의 업그레이드 시 걸림돌이다.   
이 문제에 대처하기 위한 보완 프로젝트가 "Fast FFI(foreign function interface)"와 "N-API(Node.js API)"이다.  
  
  
### Fast FFI
Fast FFI 에서는 네이티브 API를 JavaScript에 자동적으로 제공하는 것을 목표로 한다. 이것에 의해 모듈 개발자는 C/C++ 코드를 추가 작성하지 않고 기존의 네이티브 코드를 Node.js에 export 할 수 있도록 한다.  
  
Fast FFI 프로젝트는 2017년 3월 현재 초기의 프로토 타입 단계에 있으며 Node.js의 기능으로 생각되려면 작업을 더 할 필요가 있다.  
  
  
### N-API
N-API는 JavaScript VM의 네이티브 API의 ABI 안정 추상화 레이어를 제공한다.  
이로써 네이티브 모듈 개발자는 플랫폼이나 아키텍처마다 모듈을 1회만 컴파일하면 Node.js의 임의의 N-API 대응판으로 실행할 수 있게 된다.  
  
N-API는 근래 1년 동안 크게 개발이 진행되어 2017년 3월 현재는 광범위한 이용에 대응할 수 있게 되었다.  
  
가장 많이 의존하는 30개 모듈의 네이티브 API 사용 패턴에서 고찰하면 5개 이상의 모듈에서 사용되는 "V8 API"는 N-API로 100% 커버된다.  
핵심 팀은 N-API를 사용할 수 있도록 오픈 소스 "Node-Sass" "Canvas" "LevelDown" "Nanomsg" "IoTivity"를 이식했다.  
  
  
#### N-API 프로젝트의 향후 중요한 스텝은 다음과 같다.
- N-API의 Node.js 8.0 으로의 탑재를 요구하는 풀 리퀘스트를 Node.js의 마스터에 제출
- 안정화 이후 Node.js v6.9 LTS와 Node-ChakraCore에 이식
- 광범위한 커뮤니티의 약속에 의한 API 격차의 특정
- 퍼포먼스 미조정  
  
  
<br>  
  
출처: http://www.atmarkit.co.jp/ait/articles/1703/15/news069.html