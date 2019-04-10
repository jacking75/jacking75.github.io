---
layout: post
title: C# - etc
published: true
categories: [.NET]
tags: c# .net 
---
## const 배열을 정의 할 수 없을 때
  
```
static readonly int[] LEVEL_TABLE = { 1, 2, 3, 4, 5 };
static readonly string[] STR_TABLE = { "NONE", "1", "2", "3" };
```
  
  
  
## 다른 클래스에 있는 const로 정의한 상수를 이용 방법
상수는 다음과 같이 정의 되어 있다.
  
```
public class PacketDefine
{
  Public const int Packet_First = 0;
  ………..
}
```
  
이것을 Command 클래스에서 사용 할려면
  
```
Int packetfirst = PacketDefine.Packet_First;
```
  
로 사용한다.
  
  
  
## 데이터 변환
데이터 변환을 위해서는 Convert의 멤버를 사용한다.  
Convert.Toxxx( xxx )를 사용하면 된다. 예) 스트링을 int 변환  
  
```
Convert.ToInt32( string );
```
  
  
  
## 문자열이 숫자인지 검사
int_TryParse를 사용한다. 
  
```
int i = 0;
string s = "108";
bool result = int.TryParse(s, out i); //i now = 108
```
  
  
  
## 바이트 배열을 지정된 위치와 크기로 복사할 경우
  
```
Buffer.BlockCopy( ……. );
```
  
  
  
## 클래스의 크기를 알고 싶을 때
  
```
Marshal.SizeOf(LoginPacket)
```
  
LoginPacket는 인스턴스 화된 객체이어야만 한다. LoginPacket의 클래스인 LOGIN_PACKET를 사용하면 안된다.
  
  
  
## 메시지 박스
일반 메시지 박스 사용
  
```
MessageBox.Show( "클라이언트에서 사용할 스킬 정보를 파일로 저장 하겠습니까 ?" );
// YES / NO 버튼 사용
if( DialogResult.Yes == MessageBox.Show( "클라이언트에서 사용할 스킬 정보를 파일로 저장 하겠습니까 ?", "이진 파일 저장", MessageBoxButtons.YesNo ) )
```
  
  
  
## 한글 파일 출력 문제
정확한 이유는 모르지만 한글을 파일에 입력할 때 인코딩을 Default 방식으로 지정하면 한글 XP에서는 한글을 아스키 코드 값으로 제대로 인식하지만 윈도우 2003에서는 한글을 유니코드 방식으로 인식하여 한글 1글자를 아스키 코드 1글자로 인식하는 경우가 있다.
  
```
string strValue = “개”;
encodedBytes = System.Text.Encoding.Default.GetBytes(strValue);
```
  
한글 XP에서 읽는다면 encodedBytes의 길이가 2개로 나오지만 윈도우 2003에서는 길이가 1로 된다.  
이 문제를 해결할려면 인코딩 방식을 명시적으로 지정해야 된다.  
```
string strValue = “개”;
encodedBytes = System.Text.Encoding.GetEncoding(949).GetBytes(strValue);
```
  
  
  
## 한글이 포함된 Ansi 문자열을 유니코드 문자열로 변환하기
  
```
byte[] data;
string s = System.Text.Encoding.GetEncoding(949).GetString(data);
```
  
  
  
## 폼의 마우스 커서 변경
  
```
화살표 마우스 커서 this.Cursor = Cursors.Arrow;
모래시계 커서 this.Cursor = Cursors. WaitCursor;
손 커서 this.Cursor = Cursors. Hand;
기본 커서 this.Cursor = Cursors. Default;
등등…….
```
  
  
  
## 네이티브의 time(&time_t)에서 얻은 초단위의 시간을 C#에서 사용
time함수를 이용하여 얻은 시간은 시작이 1970년부터이고 C#의 경우는 0년 1월1일 부터이다..그래서 서로 호환이 되지 않느다. 이것을 해결할려면 다음과 같이 하면 된다.
  
```
int iTime = C타임함수값;
DateTime dt = new DateTime(1970, 1, 1, 9, 0, 0); // 한국은 GMT+9시간
dt = dt.AddSeconds(iTime);
```
  
  
  
## 로그 메시지 출력을 위한 가변인수와 매크로
C++에서는 로그를 남기기 위해 로그 라이브러리를 만들 때 꼭 사용하는 것이 가변인수와 매크로입니다.  
가변인수를 사용하여  
  
```
void LogMessage( LPCTSTR format, ... )
{
    char SBody[ MAX_PATH ] = { 0, };
    va_list    strlist;
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
```
  
그리고 __FILE__, __LINE__, __FUNCTION__ 매크로는 닷넷의 어트리뷰트를 사용합니다.  
  
```
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
  
  
  
## VS 출력창에 로그 출력
디버깅 시 Visual Studio의 출력 창에 로그를 남기고 싶을 때는 다음과 같이 한다.
  
```
System.Diagnostics.Debug.WriteLine(contents.ToString());
```
  
  
  
## public static bool ResetThreadPool(int? maxWorkingThreads, int? maxCompletionPortThreads)
  
```
public static bool ResetThreadPool(int? maxWorkingThreads, int? maxCompletionPortThreads, int? minWorkingThreads, int? minCompletionPortThreads)
{
   if (maxWorkingThreads.HasValue || maxCompletionPortThreads.HasValue)
   {
   }
}
```
  
  
  
## AppDomain
- 응용 프로그램에서 AppDomain 과연 어디다 쓸까? (AppDomain을 이용한 어셈블리 로딩) - 1 http://blog.naver.com/vactorman/80166702004
- 응용 프로그램에서 AppDomain 과연 어디다 쓸까? (AppDomain을 이용한 어셈블리 로딩) - 2 http://blog.naver.com/vactorman/80166705025
- AppDomain 사용 예 http://blog.naver.com/psy10604/70127422123
  
  
  
## app.config 파일을 사용하여 프로그램 설정 읽기 및 저장
- http://samurae83.blog.me/90174661488
  
```
static void PrintSettingInfo()
{
    Console.WriteLine("[ServerAddress]: {0}", ConsoleApp.Properties.Settings.Default.ServerAddress);
    Console.WriteLine("[AccountDB]: {0}", ConsoleApp.Properties.Settings.Default.AccountDB);

    var gameDBList = ConsoleApp.Properties.Settings.Default.GameDB;
    foreach (var address in gameDBList)
    {
        Console.WriteLine("[GameDB]: {0}", address);
    }

    var redisList = ConsoleApp.Properties.Settings.Default.Redis;
    foreach (var address in redisList)
    {
        Console.WriteLine("[Redis]: {0}", address);
    }
}
```
  
  
  
## 콘솔 스타일의 외부 프로세스를 실행한 후 결과얻기(혹은 윈도우의 태스크스케줄러 정보를 Linq로 분석)
- 출처: [http://d.hatena.ne.jp/kkamegawa/20101222/p1]
    - C:\>schtasks /query /XML ONE 로 태스크 스케줄러의 내용을 xml으로 출력할 수 있다.
  
```
using System;
using System.Linq;
using System.Diagnostics;
using System.Collections.Generic;
using System.IO;
using System.Xml.Linq;

namespace AvendCalender
{
  public class TaskInfo
  {
    var schTasks = new Process();

    schTasks.StartInfo.Arguments = "/query /XML ONE ";
    schTasks.StartInfo.FileName = "schtasks.exe";

    schTasks.StartInfo.UseShellExecute = false;
    schTasks.StartInfo.RedirectStandardOutput = true;
    schTasks.Start();

    string xmlText = schTasks.StandardOutput.ReadToEnd();
    XDocument xtaskXml = XDocument.Parse(xmlText);
    schTasks.WaitForExit();
    schTasks.Close();

    XNamespace xnSchTasks = "http://schemas.microsoft.com/windows/2004/02/mit/task";

    var xmlTasks = from x in xTasksList.Root.Elements(xnSchTasks + "Task")
                           select x;
  }
}
```
  
  
  
## 멀티코어 JIT 유효화 하기
- 닷넷 4.5에서 추가된 기능
- 말 그대로 멀티 코어 환경에서 병렬로 JIT을 하는 기능.
- 이것에 의해 애플리케이션의 동작에 선행하여 필요한 메소드를 JIT 할 가능성이 높아져서 결과적으로 애플리케이션 성능이 올라간다.
- ASP.NET과 실버라이트에서는 기본으로 유효화 되어 있지만 데스크탑 애플리케이션에서는 기본은 사용하지 않게 되어 있다.
** 이유는 이 기능을 사용하기 위해서는 프로파일링 처리가 필요하여 프로파일링 데이터를 보존해야 하기 때문이다. 데스크탑 애플리케이션에서는 어디에 데이터를 보존해야 할지 모르기 때문에 수동으로 해야 한다.
- 사용
  
```
ProfileOptimization.SetProfileRoot(Environment.CurrentDirectory);
ProfileOptimization.StartProfile("App.JIT.Profile");
```
  
- 참고
    - [http://d.hatena.ne.jp/gsf_zero1/20121025/p1]
    - [http://blogs.msdn.com/b/dotnet/archive/2012/10/18/an-easy-solution-for-improving-app-launch-performance.aspx]
    - [http://stackoverflow.com/questions/12965606/why-is-multicore-jit-not-on-by-default-in-net-4-5]
    - [http://msdn.microsoft.com/ja-jp/magazine/hh882452.aspx]
  
  
  
## System.Threading.Timer의 중복 콜백 호출
- System.Threading.Timer의 콜백 메소드 내에서 시간이 걸리는 처리가 있으며 콜백 매소드 처리 중에 또 다른 콜백이 호출된다.
- 예를 들면 콜백 호출 폭이 500밀리초인 경우 처리가 700밀리초 걸리는 경우 중복으로 콜백이 호출된다.
- 참고
    - [http://d.hatena.ne.jp/gsf_zero1/20120507/p1]
  
  
  
## 보안 문자 생성하기
  
```
public static int GenerateSecureNumber()
{
    var SecureNumberRandom = new Random((int)DateTime.Now.Ticks & 0x0000FFFF);
    var number = SecureNumberRandom.Next(15271493, 598410861);
    return number;
}

public static string GenerateSecureNumber2(int startNumber, int endNumber, int count)
{
    var SecureNumberRandom = new Random((int)DateTime.Now.Ticks & 0x0000FFFF);
    StringBuilder secureString = new StringBuilder();

    for( int i = 0; i < count; ++i )
    {
        secureString.Append(SecureNumberRandom.Next(startNumber, endNumber).ToString());
    }

    return secureString.ToString();
}

// 출처: http://stackoverflow.com/questions/54991/generating-random-passwords
public static string GenerateSecureString(int lowercase, int uppercase, int numerics)
{
    string lowers = "abcdefghijklmnopqrstuvwxyz";
    string uppers = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    string number = "0123456789";

    Random random = new Random();

    string generated = "!";
    for (int i = 1; i <= lowercase; i++)
        generated = generated.Insert(
            random.Next(generated.Length),
            lowers[random.Next(lowers.Length - 1)].ToString()
        );

    for (int i = 1; i <= uppercase; i++)
        generated = generated.Insert(
            random.Next(generated.Length),
            uppers[random.Next(uppers.Length - 1)].ToString()
        );

    for (int i = 1; i <= numerics; i++)
        generated = generated.Insert(
            random.Next(generated.Length),
            number[random.Next(number.Length - 1)].ToString()
        );

    return generated.Replace("!", string.Empty);

}

public static string GenerateSecureString2()
{
    Guid g = Guid.NewGuid();
    string guidString = Convert.ToBase64String(g.ToByteArray());
    guidString = guidString.Replace("=", "");
    guidString = guidString.Replace("+", "");
    return guidString;
}
```
  
  
  
## 특정 경고 무시
- 하나의 파일에서만 적용된다.
- #pragma warning disable warning-list
- #pragma warning restore warning-list
  
```
#pragma warning disable 414, 3021
[CLSCompliant(false)]
public class C
{
    int i = 1;
    static void Main()
    {
    }
}
```
  
  
  
## 값 타입에 null 대입
int, double과 같은 값 타입은 null을 대입할 수 없지만 변수 선언 시 ?을 같이 사용하면 null 대입이 가능하다.
  
```
static class 갑타입Null
{
    public static void Check()
    {
        int? y = null;

        if (y == null)
        {
            Console.WriteLine("y는 null");
            y = 11;
            Console.WriteLine("y는 {0}", y);
        }

        int? yy = GetIntValue();
        if (yy == null)
        {
            Console.WriteLine("yy는 null");
        }
    }

    static int? GetIntValue()
    {
        return null;
    }
}
```
  
  
  
## 변수 대입 시에 null 조사와 대입을 한 번에 하기
  
```
static class Null_조사와_대입을_한번에
{
    public static void Check1()
    {
        int x1 = 10;

        int x2 = x1 > 10 ? 0 : x1;
        Console.WriteLine("x2는 {0}", x2);

        x1 = 11;
        int x3 = x1 >= 10 ? 0 : x1;
        Console.WriteLine("x3는 {0}", x3);
    }

    public static void Check2()
    {
        TEST test1 = null;

        var test2 = test1 ?? new TEST() { x = 10 };
        Console.WriteLine("test2.x: {0}", test2.x);
    }

    class TEST
    {
        public int x;
    }
}
```
  
  
  
## 익명 타입
  
```
static class 익명형
{
    public static void Test()
    {
        var character = new { ID = 11, Name = "test" };
        Console.WriteLine("캐릭터 ID:{0}, Name:{1}", character.ID, character.Name);
    }
}
```
  
  
  
## const로 배열이나 오브젝트를 선언하고 싶을 때는 static readonly 사용
  
```
const int MaxValue = 100;
const Vector2 Pos = new Vector2(100, 100); // error
static readonly Vector2 Pos = new Vector2(100, 100);
```
  
  
  
## 읽기 전용 컬렉션 만들기
  
```
static class 읽기전용컬렉션
{
    static List<int> IntList = new List<int>();

    public static void Test()
    {
        IntList.Add(11);
        IntList.Add(12);

        var list = GetList();
        foreach(var value in list)
        {
            Console.WriteLine(value);
        }
    }

    static System.Collections.ObjectModel.ReadOnlyCollection<int> GetList()
    {
        return IntList.AsReadOnly();
    }
}
```
  
  
  
## 메소드 체인
- 메소드가 자신 자신의 인스턴스를 반환하는 패턴이라면 메소드 체인이 성립한다
  
```
Sample hoge = new Sample();
hoge.Piyo.Value = 100;
list.Add( hoge ); // vode를 반환
```
  
위 코드의 Add 메소드를 메소드 체인 지원으로 바꾸면 간략하게 기술 할 수 있다.  
- 확장 메소드
  
```
public static T Chain<T>( this T source , Action<T> call ) {
    if( source == null )
        return source;
    call( source );
    return source;
}
```
  
사용 예  
  
```
new Sample()
   .Chain( x => x.Piyo.Value = 100 )
   .Chain( x => list.Add( x ) );
```
  
- 출처: http://qiita.com/Temarin_PITA/items/001f71d70474063caab5
  