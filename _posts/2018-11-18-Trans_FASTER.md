---
layout: post
title: FASTER에 대해서
published: true
categories: [번역]
tags: 번역 FASTER
---
[원문](https://gist.github.com/itn3000/8811347c7066ecf02b87e008c1fb410f )  

- KVS 라이브러리
    - 내장 KVS로 사용이 가능
    - 프로세스간 스토리지 공유는 상정하지 않는 듯?
    - 메모리 이상의 DB 크기를 설정 가능(메모리 + 파일 읽기 및 쓰기)
    - LiteDB와 다소 영역이 겹칠 수도(최종 목표는 다르지만)
    - 로그 데이터베이스(내장해성) 및 인-메모리 데이터베이스(성능)의 좋은 부분을 목표로
    - 사용자 정의 update, read 콜백을 설정 가능
        - C# 버전에서는 Roslyn, C++ 버전에서는 템플릿을 사용하여 구현?
- C# 및 C++ 두 개의 구현이 존재
    - 양자 간의 성능 비교 수치는 없음
- 복제 등의 기능은 없음
- 세 종류의 스토리지 출력
    - 온 메모리 (In-Memory)
    - 추기용 로그 (Append-Only)
    - 복합(HybridLog)
- Upsert에 특히 주력
    - 동시 쓰기 스레드 수에 비례하여 처리량 향상
- 스토리지 사용량 등에 대한 비교는 없음
- C# 버전은 netcoreapp 2.1에서 움직이지 않았다
    - 실행시 컴파일에 실패
- C# 버전은 디폴트 구현은 Windows(x86 계)만 지원
    - [Windows 특정 파일 처리](https://github.com/Microsoft/FASTER/blob/master/cs/src/native/adv-file-ops/adv-file-ops.cpp )와 [Tick 취득 명령](https://github.com/Microsoft/FASTER/blob/master/cs/src/native/readtsc/readtsc.cpp )이 있고, 거기만 네이티브 처리하기 때문에
        - 파일 신장과 Tick 취득은 Linux에서도 동등한 API는 존재하기 때문에이 두 DLL을 대체 자체는 어렵지 않을듯? (rdtsc 대해 x86 의존이라는 문제는 있다)
        - [스토리지 관련 작업](https://github.com/Microsoft/FASTER/blob/master/cs/src/core/Device/LocalStorageDevice.cs )을 보면 CreateFile를 직접 호출하는 부분이 있기 때문에 결국 중간 중간 어려울 것 같은 인상
    - 사용되는 것은 주로 IDevice 구현 클래스 내(cs/core/Device 이하)이므로, 여기를 자력 구현하면 될듯?
    - 플랫폼 의존 위치와 그렇지 않은 부분의 분리는 가능할 듯(readtsc 이나 벤치 마크에서만 사용 되고 있다)
- 사용자 정의 작업을 오버 헤드 없이 수행하기 위해 C# 버전은 처음 시작할 때 대상 데이터 구조에 맞춘 처리를 코드 생성한다
    - [해당 처리 부분](https://github.com/Microsoft/FASTER/blob/master/cs/src/core/Codegen )    
- C++ 버전은 크로스 플랫폼 (x86 계만) 지원
    - Win + 32bit, Win + 64bit, Linux + 32bit, Linux + 64bit
        - Mac, xBSD 불명 (아마 안 될긋)
    - 플랫폼 마다 빌드가 필요
    - 모든 플랫폼에서 cmake가 필요
    - Win 판은 특히 VS2017 + C++ 지원이 필요
    - Linux 용 빌드의 경우 종속 라이브러리가 약간 많다
        - vcpkg 가 절대로 편할듯 
  
