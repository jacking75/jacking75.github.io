---
layout: post
title: GPU를 타깃으로 한 C# 이용
published: true
categories: [.NET]
tags: c# .net concurrent collections lock-free
---
[원문](https://www.infoq.com/news/2017/12/hybridizer)  
  
NVIDIA GPU를 이용하는 범용 프로그램을 만드는 것은 오랫동안 NVIDIA의 CUDA 플랫폼을 사용하는 것을 의미하고 있었다.  
CUDA는 몇몇 다른 프로그래밍 언어를 지원하고 있지만 고성능 코드를 기술하려면 보통 C 또는 C++ 이 필요했다.  
최근까지 이 상황에 의해서 C# 개발자는 고립되고 있으며 GPU를 타깃으로 하는 코드를 쓰기 위해서는 자신들이 좋아하는 언어를 포기할 필요가 있었다.  
  
Altimesh의 새로운 컴파일러 툴인 [Hybridizer](https://devblogs.nvidia.com/parallelforall/hybridizer-csharp/)는 이 문제에 대처했다.  
Hybridizer는 C# 개발자들에게 C# 소스에서 CUDA 바이너리를 생성함으로써 GPU를 타깃으로 하는 방법을 제공했다.  
  
Hybridizer는 2가지 버전으로 나뉘어 각각 다른 요구와 예산을 대상으로 한다.  
Hybridizer Essentials은 모두 무료이며 Visual Studio의 확장 기능으로서 제공되고 있다.  
Hybridizer Essentials는 CUDA 플랫폼의 바이너리를 생성한다.  
Hybridizer Software Suite(HSE)는 CUDA와 AVX, AVX2 및 AX512를 비롯한 다른 플랫폼을 타깃으로 하는 라이센스 소프트웨어이다.  
HSE는 바이너리를 생성하는데, 옵션으로 CUDA 소스 코드를 생성하여 사용자가 컴파일 중인 것을 감시할 수 있도록 한다.  
  
어느 옵션도 NVIDIA의 Nsight Visual Studio Edition과 조합함으로써 개발자는 Visual Studio에서 C# 코드를 기술하고 디버깅 할 수 있고, 이 결과 코드를 NVIDIA GPU에서 실행할 수 있다. HSE는 기본적으로 MSIL(Microsoft Intermediate Language) 상에서 동작하기 위해 소스 코드를 입수하지 못해도 기존 프로젝트와 통합할 수 있다. 이것에 의해 .NET 플랫폼 언어 동료인 F#과 VB.NET도 간접적으로 지원된다.  
  
CUDA 플랫폼용 C/C++ 코드를 작성하는 목표 중 하나는 최대의 성능이기 때문에 Hybridizer로 컴파일된 C# 코드의 퍼포먼스와 비교할 가치가 있다.  
Altimesh에 따르면 C#으로부터 생성된 바이너리는 CUDA를 타깃으로 한 사람이 쓴 C++ 코드의 83%의 퍼포먼스를 달성했다.  
C# 코드는 몇가지 실천적인 관여로 C++ 코드의 퍼포먼스와 비슷한 수준으로 개선됐다.  
  
Hybridizer 소프트웨어로 CUDA와 GPU 프로그래밍에 관심이 많은 C# 개발자는 자신이 좋아하는 기술을 버리지 말고 이들 기술을 탐구할 수 있다.  
샘플 코드는 GitHub에서 입수할 수 있고 Hybridizer Essentials의 확장 기능은 Visual Studio Marketplace에서 입수 할 수 있다.  
   