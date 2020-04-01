---
layout: post
title: Rust - bytes Read, Write
published: true
categories: [Rust]
tags: rust bytes
---
[출처](https://keens.github.io/blog/2016/12/01/rustdebaitoretsuwoatsukautokinotips/  )  
  
## Read 와 Write
  
```  
let mut bytes: &[u8] = &[1, 2, 3, 4, 5, 6];
let mut buf = [0;3];
bytes.read_exact(&mut buf).unwrap();
println!("read: {:?}, rest: {:?}", buf, bytes);
```
  
결과  
<pre>
read: [1, 2, 3], rest: [4, 5, 6]
</pre>
  
<br>  
  
```
let bytes: Vec<u8> = vec![1, 2, 3, 4, 5, 6];
let mut buf = [0;3];
(&bytes[..]).read_exact(&mut buf).unwrap();
println!("read: {:?}, rest: {:?}", buf, bytes);
```
  
결과  
<pre>
read: [1, 2, 3], rest: [1, 2, 3, 4, 5, 6]
</pre>
  
<br>   
   
첫 번째 코드처럼 하고 싶다면 아래처럼 한다  
```
let bytes: Vec<u8> = vec![1, 2, 3, 4, 5, 6];
let mut bytes = &bytes[..];
let mut buf = [0;3];
bytes.read_exact(&mut buf).unwrap();
println!("read: {:?}, rest: {:?}", buf, bytes);
```
  
<br>  
  
Write도 비슷함. 이 대로 write 가능하고, 소비된다. 쓰는 영역이 비어도 그대로 넘어가므로 주의해야한다.  
```
let mut bytes: &mut [u8] = &mut [0; 6];
let data = &[1, 2, 3];
println!("data: {:?}, buf: {:?}", data, bytes);
bytes.write(data).unwrap();
println!("data: {:?}, buf: {:?}", data, bytes);
bytes.write(data).unwrap();
println!("data: {:?}, buf: {:?}", data, bytes);
bytes.write(data).unwrap();
println!("data: {:?}, buf: {:?}", data, bytes);
```
  
결과  
<pre>
data: [1, 2, 3], buf: [0, 0, 0, 0, 0, 0]
data: [1, 2, 3], buf: [0, 0, 0]
data: [1, 2, 3], buf: []
data: [1, 2, 3], buf: []
</pre>
  
<br>   
  
마지막에 버퍼가 빈 상태로 쓰기를 해도 별다른 에러 없이 끝난다. 그리고 쓴 데이터로의 접근은 불가능하다.  
아래처럼 바꾼다.    
```
let mut bytes:[u8; 6] =  [0; 6];
let data = &[1, 2, 3];
{
    let mut buf: &mut [u8] = &mut bytes;
    println!("data: {:?}, buf: {:?}", data, buf);
    buf.write(data).unwrap();
    println!("data: {:?}, buf: {:?}", data, buf);
    buf.write(data).unwrap();
    println!("data: {:?}, buf: {:?}", data, buf);
}
println!("bytes: {:?}", bytes);
```
  
결과  
<pre>
data: [1, 2, 3], buf: [0, 0, 0, 0, 0, 0]
data: [1, 2, 3], buf: [0, 0, 0]
data: [1, 2, 3], buf: []
bytes: [1, 2, 3, 1, 2, 3]
</pre>
  
<br>     
  
Vec을 사용하면 더 좋다. 가변 길이도 Write 가능하다.  
```
let mut bytes: Vec<u8> = Vec::new();
let data = &[1, 2, 3];
println!("data: {:?}, buf: {:?}", data, bytes);
bytes.write(data).unwrap();
println!("data: {:?}, buf: {:?}", data, bytes);
bytes.write(data).unwrap();
println!("data: {:?}, buf: {:?}", data, bytes);
println!("bytes: {:?}", bytes);
```
  
결과  
<pre>
data: [1, 2, 3], buf: []
data: [1, 2, 3], buf: [1, 2, 3]
data: [1, 2, 3], buf: [1, 2, 3, 1, 2, 3]
bytes: [1, 2, 3, 1, 2, 3]
</pre>
  
  
<br>  
  
  
## Cursor
std::io::Cursor를 사용하면 좀 더 쉽게 할 수 있다.  
Cursor는 생성자에서 인수의 소유권을 박탈하는 랩퍼 오브젝트적 구조체이다.  
  
read의 예  
```
let bytes: &[u8] = &[1,2,3,4,5,6];
let data: &mut [u8] = &mut [0;3];
let mut cur = Cursor::new(bytes);
println!("data: {:?}, position {}", data, cur.position());
cur.read_exact(data).unwrap();
println!("data: {:?}, position {}", data, cur.position());
cur.read_exact(data).unwrap();
println!("data: {:?}, position {}", data, cur.position());
println!("bytes: {:?}", cur.into_inner());
```
  
결과  
<pre>
data: [0, 0, 0], position 0
data: [1, 2, 3], position 3
data: [4, 5, 6], position 6
bytes: [1, 2, 3, 4, 5, 6]
</pre>
  
<br>   
  
Write의 예이다.  
```
let bytes: &mut [u8] = &mut [0;6];
let data: &[u8] = &[1, 2, 3];
let mut cur = Cursor::new(bytes);
println!("bytes: {:?}, position {}", cur.get_ref(), cur.position());
cur.write(data).unwrap();
println!("bytes: {:?}, position {}", cur.get_ref(), cur.position());
cur.write(data).unwrap();
println!("bytes: {:?}, position {}", cur.get_ref(), cur.position());
println!("bytes: {:?}", cur.into_inner());
```
  
결과  
<pre>
data: [0, 0, 0, 0, 0, 0], position 0
data: [1, 2, 3, 0, 0, 0], position 3
data: [1, 2, 3, 1, 2, 3], position 6
bytes: [1, 2, 3, 1, 2, 3]
</pre>
  
<br>     
  
또 Cursor의 재미있는 점으로 &[u8] 가 아닌 AsRef<[u8]>로 Read를 구현하고 있고, std::io::Seek도 구현하고 있다.  
```
let bytes: Vec<u8> = Vec::new();
let data: &[u8] = &[1, 2, 3];
let buf: &mut [u8] = &mut [0; 3];
let mut cur = Cursor::new(bytes);
println!("bytes: {:?}, position {}", cur.get_ref(), cur.position());
cur.write(data).unwrap();
println!("bytes: {:?}, position {}", cur.get_ref(), cur.position());
cur.write(data).unwrap();
println!("bytes: {:?}, position {}", cur.get_ref(), cur.position());
cur.seek(SeekFrom::Start(0)).unwrap();
println!("bytes: {:?}, position {}", cur.get_ref(), cur.position());
cur.read(buf).unwrap();
println!("bytes: {:?}, position {}, buf: {:?}", cur.get_ref(), cur.position(), buf);
```
  
결과  
<pre>
bytes: [], position 0
bytes: [1, 2, 3], position 3
bytes: [1, 2, 3, 1, 2, 3], position 6
bytes: [1, 2, 3, 1, 2, 3], position 0  // <- 0 으로 옮겼다
bytes: [1, 2, 3, 1, 2, 3], position 3, buf: [1, 2, 3] // <- 0에서 read 가능하다
</pre>
  
  