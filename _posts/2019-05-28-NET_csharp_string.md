---
layout: post
title: C# - 문자열
published: true
categories: [.NET]
tags: c# .net string
---
## 지정 단어로 문자열 분해
  
```
string readLine = "A=B";
string[] word = readLine.Split(new Char[] { '=' });
```
  
  
  
## 문자열 포맷 
  
```
string CheatString = string.Format("@timeevent1:{0}", textBoxBlessing.Text);
```
  
  
  
## 지정된 단어를 문자열에서 (제일 처음 나오는)제거 하고 싶을 때
  
```
string str1 = "C:\ASDD";
string str2 = "C:\ASDD\Server.exe";
// str2에서 str1을 제거하고 싶음
string str3 = str2.Trim( str1.ToCharArray() );
``` 
  
  
  
## 문자열 "C\Text\Sub" 에서 "Sub"만 추출 하고 싶을 때
  
```
textBox1.Text = "C\Text\Sub";
int index = textBox1.Text.LastIndexOf(@"\");
string CurVersionFoldName = textBox1.Text.Substring(index + 1);
``` 
  
  
  
### 바이트 길이 알기
  
```
int iAsciiLength = ((byte[])Encoding.GetEncoding(0).GetBytes(stringData)).Length;  
```
  