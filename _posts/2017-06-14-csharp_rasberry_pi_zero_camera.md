---
layout: post
title: rasberry pi zero + C#으로 카메라 조작
published: true
categories: [.NET]
tags: c# .net rasberry iot
---
nuget의 아래 모듈을 받는다.
https://www.nuget.org/packages/Unosquare.Raspberry.IO/  
GPIO 에 더해서 카메라 모듈도 조작할 수 있다.  
이것을 사용하여

```
public static async Task SendImage()
{
    var cam = new Unosquare.RaspberryIO.Camera.CameraController();
    var result = await cam.CaptureImageJpegAsync(640, 480,System.Threading.CancellationToken.None);
}
```

 
    
<br>
<br>  

출처: http://qiita.com/divideby_zero/items/9bb550529d539a054dfc