---
layout: post
title: C# - C++과 연동
published: true
categories: [.NET]
tags: c# .net c++ marshal
---
꼭 봐야 될 것   
- http://sj21.wo.to/tt/483  
- http://sj21.wo.to/tt/484   
- http://blogs.msdn.com/junfeng/archive/2006/05/20/599434.aspx  
  
  
## How to: Marshal Structures Using C++ Interop How to: Marshal Embedded Pointers Using C++ Interop  
C++  
```
#include <msclr\marshal.h>   
   
using namespace System;   
   
using namespace msclr::interop;   
   
int main(int argc, char** argv)   
{   
    const char* x = "Mixing C++ and C#";   
    String^ y;   
    y = marshal_as<String^>(x);   
    return 0;   
}
```
  
Overview of Marshaling in C++ http://msdn.microsoft.com/en-us/library/bb384865.aspx   
C++에서 C#의 함수를 콜백으로 받아서 처리  
C++  
```
typedef void (__stdcall *PROGRESS)(int);   
   
class CalcCPP   
{   
    int Add(PROGRESS callBack, int a, int b)   
    {   
        int result = 0;   
        for(int i = 0; i < 100; ++i)   
        {   
            c = (a + b);   
            callBack(i);   
        }   
        return c;   
    }   
}
```  
    
  
C#에서 C++의 함수를 함수 콜백으로 사용 C++/CLR  
C++/CLI  
```
using namespace System;   
using namespace System.Runtime.InteropServices;   
   
namespace Test   
   
{   
    public delegate void PROGRESS_CSHARP(int);   
   
    public ref class CalcWrapper   
   
    {   
    private:   
        CalcCPP* m_pCpp;           
   
    public:   
        CalcWrapper()    
        {m_pCpp = new CalcCPP()}   
        virtual ~CalcWrapper()    
        {if(m_pCpp){delete m_pCpp; m_pCpp = 0;}}   
   
        int Add(PROGRESS_CSHARP^ callBackFromCSharp,    
                  int a, int b)   
        {   
            // GC를 할당한 뒤   
            GCHandle gch =    
                GCHandle::Alloc(callBackFromCSharp);   
            //  delegate를 함수 포인터로 만듭니다.   
            IntPtr ip =    
                Marshal::GetFunctionPointerForDelegate(   
                     callBackFromCSharp   
                     );   
            // 그리고 형변환해서 대입   
            PROGRESS callBack =    
                static_cast<PROGRESS>(ip.ToPointer());   
   
            // 사용!   
            int result = m_pCpp->Add(callBack, a, b);   
   
            // 할당했던 GC를 비우면서 해제합니다   
            GC::Collect();   
            gch.Free();   
   
            return result;   
        }   
   
    }   
}
```  
  
C#에서 사용할 때는  
C#  
```
class TestClass   
{   
    static void Progress(int pos)   
    {   
        System.Console.WriteLine(   
            "Currently : " + pos.ToString()   
            );   
    }   
   
    static void Main(string[] args)   
    {   
        Test.CalcWrapper cw = new Test.CalcWrapper();   
        int result = cw.Add(Progress, 1, 2);   
        System.Console.WriteLine(   
            "Result is " + result.ToString()   
            );   
    }   
}
```  
  
  
  
## C/C++ 구조체와 함수를 C# 에서 사용하기
출처: http://hado.hadostudio.com/blog/category/DotNet  
기존의 C/C++ 의 구조체와 함수들을 C# 의 클래스로 캡슐화하는 작업을 설명.  
먼저, C/C++ 에서 DLL 프로젝트를 생성한다. (프로젝트 이름을 'BND.Native' 라고 하면 출력물은 'BND.Native.dll' 이 된다.)  

### 1단계: C/C++ 구조체 정의 (MD5.h)  
C++  
```
typedef struct mD5Context
{
   UINT state[4];   /* state (ABCD) */
   UINT count[2];   /* number of bits, modulo 2^64 (lsb first) */
   BYTE buffer[64]; /* input buffer */
}
 
MD5Context;
```
  
구조체 내부에 고정 크기의 배열이 정의되어 있다.  
  
  
  
### 2단계: C/C++ 함수 정의 (MD5.cpp)  
C++  
```
void MD5Init(MD5Context * context)
{
   ...
}
 
void MD5Update(MD5Context * context, BYTE * input, UINT input_size)
{
  ...
}
 
void MD5Final(MD5Context * context, BYTE * output_digest)
{
   ...
}
```  
C/C++ 언어답게(?) 포인터를 마구 사용하고 있다.  
  
  
### 3단계: C/C++ 함수 선언 (MD5.h)  
C++  
```
extern "C" __declspec(dllexport) void MD5Init(MD5Context * context);
extern "C" __declspec(dllexport) void MD5Update(MD5Context * context, BYTE * input, UINT input_size);
extern "C" __declspec(dllexport) void MD5Final(MD5Context * context, BYTE * output_digest);
```
  
함수들을 DLL 외부로 노출한다.  
이를 위해서 #include <Windows.h> 라인을 추가해야 한다.  
이상의 3단계를 작업한 후 컴파일하여 C/C++ DLL 을 생성한다.  
다음으로 C# 프로젝트를 생성한다.  
C# 프로그램에서 C/C++ DLL 을 참조하기 위해 [C# 프로젝트의 속성] - [빌드 이벤트] - [빌드 후 이벤트 명령줄] 에 다음을 추가한다.  
COPY /Y "$(SolutionDir)BND.Native\$(ConfigurationName)\BND.Native.dll" "$(TargetDir)BND.Native.dll"  
C/C++ DLL 파일을 C# 프로젝트의 출력 디렉토리로 복사해 오는 것이다.  
이를 위해서 프로젝트 '종속성'에 'BND.Native' 프로젝트를 추가해야 한다.  
  
  
  
### 4단계: 구조체 캡슐화  
C#  
```
public class MD5Calculator
{
 
    // C/C++ 의 구조체와 동일한 형태가 되도록 정의합니다.
    public struct MD5Context
    {
        [MarshalAs(UnmanagedType.ByValArray, SizeConst =  4)] public UInt32[] state;
        [MarshalAs(UnmanagedType.ByValArray, SizeConst =  2)] public UInt32[] count;
        [MarshalAs(UnmanagedType.ByValArray, SizeConst = 64)] public byte[] buffer;
    }
 
    // 클래스 내부에 구조체 인스턴스를 캡슐화합니다.
    private MD5Context mContext = new MD5Context();
 
    // 구조체를 초기화하는 코드를 생성자 내부에 작성합니다.
    public MD5Calculator()
    {
        /* 따로 new 하지 않아도 이미 공간이 확보되어 있음.
        mContext.state = new UInt32[4];
        mContext.count = new UInt32[2];
        mContext.buffer = new byte[64];
        */
    }
```  
  
C/C++ 에서 정의한 구조체와 동일한 구조체를 정의하고, 그 인스턴스를 정의하고, 초기화 코드를 작성하였다.  
이 작업들을 편의상 하나의 클래스(MD5Calculator) 내에 캡슐화하였다.  
  
  
### 5단계: 함수 참조 선언  
C#  
```
// C언어 함수들
[DllImport("BND.Native.dll")] extern public static void MD5Init(ref MD5Context ctx);
[DllImport("BND.Native.dll")] extern public static void MD5Update(ref MD5Context ctx, byte[] input, int input_size);
[DllImport("BND.Native.dll")] extern public static void MD5Final(ref MD5Context ctx, byte[] output_digest);
```  
  
각종 포인터 파라미터가 위와 같이 매핑된다.  
  
  
### 6단계: 함수 캡슐화  
C#  
```
public void Initial()
{
    MD5Init(ref mContext);
}
 
public void Update(byte[] input, int input_size)
{
   MD5Update(ref mContext, input, input_size);
}
 
public byte[] Final()
{
    byte[] output_digest = new byte[16];
    MD5Final(ref mContext, output_digest);
    return output_digest;
}
```  
  
구조체를 캡슐화 했듯이, 함수들도 캡슐화해 준다.  
  
C#에서 C++ 함수로 포인터 배열 넘기기 혹시 저와 같은 고민을 하시는 분들이 보시고 조금이라도 도움이 되기를 바라며 1주일 동안 고생하며 스스로 찾아낸 답들을 올려보겠다.   
  
C++  
```
void CalOutputs(float* ivec);
float* GetOutput(void);
void GetOutput(float* ovec);
void TeachOnce(float* ivec, float* ovec, float Rate, float momentum);
```  
  
위의 함수들은 예전에 작성된 C++ 클래스의 멤버 메쏘드이다.   
모두 포인터 배열을 넘기고 받기 때문에 C#에서는 그대로 사용하기 어려운 면이 있어 C# wrapper class를 만들고 다음과 같이 메쏘드를 wrapping 했다.  
  
배열 넘기기  
C++  
```
public void CalOutputs(float[] ivec)
{
    unsafe
    {
        fixed(float* _ivec = ivec)
        {
            this.bpNet.CalOutputs(_ivec);
        }
    }
}
```
  
배열 리턴 값 받기  
특히 이 부분이 어려웠는데, 배열을 바로 리턴 받을 방법은 없고 그렇다고 마셜링을 한다든가 하면 퍼포먼스에 문제가 있을 거 같아서 원본 C++ 클래스에는 float* GetOutput(void); 으로 되어 있던 메쏘드를 void GetOutput(float* ovec); 로 추가하여 wrapping 했다.  
만일 소스코드를 바로 다룰 수가 없는 DLL 상태라든가 하면 DLL 과 C# 사이에 이 wrapping 해주는 메쏘드를 포함하는 C++ wrapper 클래스를 추가해주면 된다.  
  
C#  
```
public float[] GetOutput()
{
    float[] result = new float[LENGTH];
 
    unsafe
    {
        fixed(float* _result = result)
        {
            this.bpNet.GetOutput(_result);
        }
    }
 
    return result;
}
```
  
같은 타입의 배열을 두 개 이상 넘길 때  
  
C#  
```
public void TeachOnce(float[] ivec, float[] ovec, float Rate, float momentum)
{
    unsafe
    {
 
        fixed(float* _ivec = ivec, _ovec = ovec)
        {
            this.bpNet.TeachOnce(_ivec, _ovec, Rate, momentum);
        }
    }
}
```
  