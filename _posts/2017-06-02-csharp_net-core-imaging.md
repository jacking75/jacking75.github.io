---
layout: post
title: .NET Core용 이미지 라이브러리
published: true
categories: [.NET]
tags: c# .net rasberry
---
.NET Core를 사용하여 애플리케이션을 개발할 때 개발자가 알아야 할 결점 하나는 영상 기반 API가 없는 것이다.  
보급된 API 중의 하나로 훌륭한 System.Drawing 이 있지만 이것은 Windows 기반의 GDI+ 인터페이스에 의존하고 있다.  
.NET Core에서는 이용할 수 없다.  
다행스럽게도 많은 개발자 커뮤니티가 .NET Core를 지원하는 영상 라이브러리를 만들어서 대처 하고 있다.  
  
Microsoft의 Bertrand Le Roy씨는 각 라이브러리의 조사와 비교를 하여 이것들의 적합성을 평가했다.  
처음 조사는 이들 4개의 구현 아웃풋으로 퍼포먼스를 비교하기 위한 유용한 기준을 제시하고 있다.

- CoreCompat.System.Drawing
- ImageSharp
- Magick.NET
- SkiaSharp  
  
  
System.Drawing에 의존하고 있는 코드의 개발자는 CoreCompat.System.Drawing 라이브러리가 일시적인 대체용으로 유용함을 알 수 있다. 다만, Windows에서 실행할 경우 잠금 문제가 발생할 수 있다.  
  
  
ImageSharp은 순수 매니지 코드로 만들어진 새 라이브러리이다. 
이에 따른 성능 향상을 위해 네이티브 코드를 사용하지 않아서 크로스 플랫폼 지원이 대폭 향상된다.
  
  
Magick.NET은 오픈 소스 ImageMagick 라이브러리용의 .NET 기반의 래퍼이다.  
이것은 많은 기능을 제공하고 Le Roy씨가 최고의 화상 품질이라고 생각하는 것들을 만들어 내지만 현재는 Windows 상에서의 .NET Core만을 지원하고 있다.  
다른 플랫폼의 지원에 대해서 Magick.NET의 저자(Dirk Lemstra씨)는 ImageMagick 자체를 크로스 플랫폼으로서 제공하고 싶어 한다.  
  
  
SkiaSharp에는 Google의 Skia 크로스 플랫폼의 2D 그래픽 라이브러리용 .NET 래퍼가 있지만 .NET Core는 지원하지 않는다.   
[Miguel de Icaza씨는 .NET 코어를 지원하기 위한 과제에 대해서 설명했다.](https://github.com/NuGet/Home/issues/3114)  
  
<br>  
    
Le Roy씨는 결론으로 어떤 라이브러리가 현재의 개개의 요구에 가장 적합한지를 말했다.  
CoreCompate.System.Drawing은 lock의 문제가 허용되는 어플리케이션이라고 가정했을 경우 높은 성능을 발휘한다.  
Magick.NET은 최고의 품질과 파일 유형을 지원하고 있다.  
마지막으로 ImageSharp은 순수 매니지 코드로 성능은 다른 라이브러리에 뒤지지만 적극적인 개발이 진행되고 있어 조만간 보다 적절한 최적화가 가능하다.
 
    
<br>
<br>  

출처: https://www.infoq.com/news/2017/01/net-core-imaging