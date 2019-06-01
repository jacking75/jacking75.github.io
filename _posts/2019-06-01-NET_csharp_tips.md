---
layout: post
title: C# - tip 모음
published: true
categories: [.NET]
tags: c# .net
---
## System.BitConverter
  
### byte에서 short로 변환하기
http://msdn.microsoft.com/ko-kr/library/system.bitconverter.toint16.aspx  
BitConverter.ToInt16   
  
  
### 스트링을 바인트로 변환 
  
```
//the below code converts byte[] type string type   
string txt = System.Text.Encoding.GetEncoding("utf-8").GetString(bytes);  

//using Default Encoding
string txt = System.Text.Encoding.Default.GetString(msg2.getData());
```
  
  
  
## 다른 클래스에 있는 const로 정의한 상수를 이용 방법
  
```
// 상수는 다음과 같이 정의 되어 있다.
public class PacketDefine
{
  Public const int Packet_First = 0;
  ………..
}
```
  
```
//이것을 Command 클래스에서 사용 할려면
{
  // …………
  int packetfirst = PacketDefine.Packet_First;
  // ………….
}
```  
   
  
  
### 데이터 변환
데이터 변환을 위해서는 Convert의 멤버를 사용한다.  
```
// Convert.Toxxx( xxx )를 사용하면 된다. 예) 스트링을 int 변환
{
  Convert.ToInt32( string );
} 
```  
  
  
  
### 바이트 배열을 지정된 위치와 크기로 복사할 경우  
  
```
{
  Buffer.BlockCopy( ……. );
}
```  
    
  
  
### 클래스의 크기를 알고 싶을 때  
  
```
{
  Marshal.SizeOf(LoginPacket)
}
//LoginPacket는 인스턴스 화된 객체이어야만 한다. LoginPacket의 클래스인 LOGIN_PACKET를 사용하면 안된다.
```
  
  
  
### 한글 파일 출력 문제  
정확한 이유는 모르지만 한글을 파일에 입력할 때 인코딩을 Default 방식으로 지정하면 한글 XP에서는 한글을 아스키 코드 값으로 제대로 인식하지만 윈도우 2003에서는 한글을 유니코드 방식으로 인식하여 한글 1글자를 아스키 코드 1글자로 인식하는 경우가 있다.  
```
{
  string strValue = “개”;
  encodedBytes = System.Text.Encoding.Default.GetBytes(strValue);
}
```  
  
한글 XP에서 읽는다면 encodedBytes의 길이가 2개로 나오지만 윈도우 2003에서는 길이가 1로 된다.  
이 문제를 해결할려면 인코딩 방식을 명시적으로 지정해야 된다.  
```
{
  string strValue = “개”;
  encodedBytes = System.Text.Encoding.GetEncoding(949).GetBytes(strValue);
}
```  
  
  
  
### 한글이 포함된 Ansi 문자열을 유니코드 문자열로 변환하기
  
```
{
  byte[] data;
  string s = System.Text.Encoding.GetEncoding(949).GetString(data);
}
```   
  
  
  
### 네이티브의 time(&time_t)에서 얻은 초단위의 시간을 C#에서 사용
time함수를 이용하여 얻은 시간은 시작이 1970년부터이고 C#의 경우는 0년 1월1일 부터이다..그래서 서로 호환이 되지 않느다. 이것을 해결할려면 다음과 같이 하면 된다.  
```
{
  int iTime = C타임함수값;    
  DateTime dt = new DateTime(1970, 1, 1, 9, 0, 0); // 한국은 GMT+9시간
  dt = dt.AddSeconds(iTime);
}
```
  
  
  
### 로그 메시지 출력을 위한 가변인수와 매크로
C++에서는 로그를 남기기 위해 로그 라이브러리를 만들 때 꼭 사용하는 것이 가변인수와 매크로입니다.   
```
//가변인수를 사용하여
void LogMessage( LPCTSTR format, ... )
{
  char SBody[ MAX_PATH ] = { 0, };
  va_list  strlist;
  va_start( strlist, format );
  _vsntprintf_s( SBody, _countof(SBody), _TRUNCATE, format, strlist );
  va_end( strlist );
}
```  
이런 식으로 하나의 메시지를 만들고   
이 LogMessage함수를 어디서, 언제 호출했는지 알리는 방법은 __FILE__, __LINE__, __FUNCTION__ 와 같은 매크로를 사용합니다.   
  
이런 기능을 닷넷에서 구현하려면 가변인수는 다음과 같은 방식으로 구현 합니다.  
```
public void print(int i, params string[] messages)
{
    string logmsg;
    
    foreach(string msg in messages)
        logmsg += msg;

}

// 그리고 __FILE__, __LINE__, __FUNCTION__ 매크로는 닷넷의 어트리뷰트를 사용합니다.
using System.Runtime.CompilerServices;
using System.Diagnostics;

void LogToUI(string logText,
                    [CallerFilePath] string fileName = "",
                    [CallerMemberName] string methodName = "",
                    [CallerLineNumber] int lineNumber = 0)
{
    string logmsg = string.Format("[줄:{0}|호출:{1}|시간{2}] : {3}", lineNumber, methodName, DateTime.Now.ToShortTimeString(), logText);
    listBoxLog.Items.Add(logmsg);
}

// 사용 
LogToUI("서버 생성 성공");
```
  