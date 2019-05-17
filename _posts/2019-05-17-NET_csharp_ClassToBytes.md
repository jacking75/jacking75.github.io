---
layout: post
title: C#의 클래스를 byte[]로 변환하는 방법
published: true
categories: [.NET]
tags: c# .net classToBytes
---
  
패킷 헤더 클래스  
```
[StructLayout(LayoutKind.Sequential)]//[StructLayout(LayoutKind.Sequential, Pack=1)]
public class HEADER
{
    public ushort a1;
    public ushort a2;
    public ushort a3;
    public ushort a4;
}
```
  
  
로그인 요청 클래스  
```
// GetBuffer을 부모 클래스에서 정의하고 여기서는 상속 받지 않은 이유는 그렇게 하면 클래스의
// 데이타를 복사 할 때 부모클래스의 크기(4바이트) 만큼을 앞에 계산 해버린다
[StructLayout(LayoutKind.Sequential)]
public class LoginAuthorRet
{
    public LoginAuthorRet()
   {
        Header = new HEADER();
        acID = new byte[21];
        acPasswd = new byte[31];
    }

    // 클래스의 있는 데이타를 메모리에 담아서 리턴 한다.
    public void GetBuffer( byte[] outBuffer )
    {
        if( 0 == outBuffer.Length )
            outBuffer = new byte[ MAX_PACKET_DATA ];

        unsafe
       {
            fixed(byte* fixed_buffer = outBuffer)
           {
                Marshal.StructureToPtr(this, (IntPtr)fixed_buffer, false);
            }
        }
    }

    public HEADER Header;    // 헤더
    [MarshalAs(UnmanagedType.ByValArray, SizeConst=21)] public byte[] acID;          // 아이디
   [MarshalAs(UnmanagedType.ByValArray, SizeConst=31)] public byte[] acPasswd;    // 패스워드
}
```
  
   
사용 예  
```
LoginAuthorRet LoginPacket = new LoginAuthorRet();
LoginPacket.Header.a2 = PK_LOGIN_AUTHOR_REQ;
LoginPacket.Header.a3 = (ushort)(Marshal.SizeOf(LoginPacket)-Marshal.SizeOf(LoginPacket.Header));
               
int iIDLen = strID.Length;
int iPWLen = strPass.Length;
Buffer.BlockCopy( Encoding.ASCII.GetBytes(strID), 0, LoginPacket.acID, 0, iIDLen );
Buffer.BlockCopy( Encoding.ASCII.GetBytes(strPass ), 0, LoginPacket.acPasswd, 0, iPWLen );

byte[] packet1 = new byte[ MAX_PACKET_DATA ];
LoginPacket.GetBuffer( packet1 );
SendPacket( packet1, Marshal.SizeOf(LoginPacket) );
```
  