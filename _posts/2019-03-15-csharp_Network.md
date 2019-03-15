---
layout: post
title: C# - 자신의 ip 주소 얻기
published: true
categories: [.NET]
tags: c# .net network
---
  
```
string myIPAddress = "";
var ipentry = System.Net.Dns.GetHostEntry(System.Net.Dns.GetHostName());

foreach (var ip in ipentry.AddressList)
{
    if (ip.AddressFamily == System.Net.Sockets.AddressFamily.InterNetwork)
    {
        myIPAddress = ip.ToString();
        break;
    }
}
```
  

