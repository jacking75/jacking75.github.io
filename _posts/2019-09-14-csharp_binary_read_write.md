---
layout: post
title: C# - binary 데이터 읽고, 쓰기
published: true
categories: [.NET]
tags: .net c# .netcore binary
---
## 읽기
  
### BitConverter 
BitConverter 클래스의 타입 별 함수를 사용한다  
```
var data = new byte[2048];
var pos = 0;

Int16 PacketID = BitConverter.ToInt16(data, pos);
pos += 2;
```
  
  
### UnSafe
  
```  
public static unsafe short Int16(byte[] bytes, int offset)
{
    fixed (byte* ptr = bytes)
    {
        return *(short*)(ptr + offset);
    }
}  
```
  
  
  
## 쓰기
  
### List<byte>  
  
```
public byte[] ToBytes()
{
    Int16 MsgLen = 100;
    Int32 TextLen = 324324;
    
    List<byte> dataSource = new List<byte>();
    dataSource.AddRange(BitConverter.GetBytes(MsgLen));
    dataSource.AddRange(BitConverter.GetBytes(TextLen));
    return dataSource.ToArray();
}
```
  
  
### BinaryWriter
  
```
var ms = new System.IO.MemoryStream(sendData); 
using (var bw = new System.IO.BinaryWriter(ms))
{
    bw.Seek(3, System.IO.SeekOrigin.Begin); // 3바이트 이동
    bw.Write((UInt16)sendData.Length);
    bw.Write((UInt16)PACKET_ID.PACKET_ID_ROOM_CHAT_REQ);
}
```
  
  
### UnSafe
  
```
public static unsafe int Int16(byte[] bytes, int offset, short value)
{
    fixed (byte* ptr = bytes)
    {
        *(short*)(ptr + offset) = value;
    }

    return 2;
}
```
  