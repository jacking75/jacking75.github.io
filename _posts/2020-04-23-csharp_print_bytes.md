---
layout: post
title: C# - byte array 내용 출력하기
published: true
categories: [.NET]
tags: .net c# .netcore bytes
---
네트워크 프로그래밍을 할 때 소켓을 통해서 받은 데이터의 바이너리 값을 보고 싶을 경우가 있다.  
바로 출력은 안되고 아래의 [스택오버플로우의 글](https://stackoverflow.com/questions/10940883/c-converting-byte-array-to-string-and-printing-out-to-console )을 사용하면 좋다.  
  
```  
static public string ToReadableByteArray(byte[] bytes)
{
    return string.Join(", ", bytes);
}
```
  
```
Console.WriteLine(ToReadableByteArray(bytes));
```
  
혹은  
<br>  
  
  
```
public void PrintByteArray(byte[] bytes)
{
    var sb = new StringBuilder("new byte[] { ");
    foreach (var b in bytes)
    {
        sb.Append(b + ", ");
    }
    sb.Append("}");
    Console.WriteLine(sb.ToString());
}
```
  
```
var signedBytes = new sbyte[] { 1, 2, 3, -1, -2, -3, 127, -128, 0, };
var unsignedBytes = UnsignedBytesFromSignedBytes(signedBytes);
PrintByteArray(unsignedBytes);
// output:
// new byte[] { 1, 2, 3, 255, 254, 253, 127, 128, 0, }
```
  
```
// http://stackoverflow.com/a/829994/346561
public static byte[] UnsignedBytesFromSignedBytes(sbyte[] signed)
{
    var unsigned = new byte[signed.Length];
    Buffer.BlockCopy(signed, 0, unsigned, 0, signed.Length);
    return unsigned;
}
```
  