---
layout: post
title: C# - ArraySegment 배열을 새로 할당하지 않고 특정 위치의 내용을 참고 하고 싶을 때
published: true
categories: [.NET]
tags: c# net ArraySegment
---
- 배열(예를들면 네트워크에서 받은 데이터를 저장한 버퍼 byte[] buffer)의 특정 위치에서 특정 크기만큼 참고하고 싶을 때 보통 새로 배열을 만든 후 복사해야 원하는 데이터만을 참조할 수 있다.
- 그러나 ArraySegment를 사용하면 새로 배열을 만들지 않으면서 버퍼의 특정 데이터를 참조할 수 있다.
- ArraySegment를 사용하여 1차원 배열의 랩퍼로 배열 내의 요소를 범위(시작 위치와 길이)를 지정하여 구별한다.
  
```
int[] ary1 = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// 배열을 나눈다. ary1의 범위만큼의 참조를 만든다.
ArraySegment<int> arySeg = new ArraySegment<int>(ary1, 2, 5);

// 나누어진 범위의 요소를 출력한다.
for (int i = arySeg.Offset; i < arySeg.Offset + arySeg.Count; i++)
{
	Console.WriteLine("{0}:{1}", i, arySeg.Array[i]);
}
```
  
아래와 같이 표시된다  
<pre>
2:2
3:3
4:4
5:5
6:6
</pre>
  
<br/>
  
arySeg.Array은 ary1과 같으므로 배열을 사용하는 곳에 ArraySegment를 사용하기 위해서는 아래처럼 한다.  
```
// ArraySegment<byte> recvData

var packetID = BitConverter.ToInt32(recvData.Array, recvData.Offset);

var packetBody = new byte[recvLength];
Buffer.BlockCopy(recvData.Array, (recvData.Offset+12), packetBody, 0, (recvLength - 12));
```
  