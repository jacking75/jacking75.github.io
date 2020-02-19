---
layout: post
title: Python - struct 모듈을 사용하여 Binary 읽고/쓰기
published: true
categories: [Python]
tags: python binary
---
[출처](https://www.atmarkit.co.jp/ait/articles/1910/01/news020_3.html  )  
  
## struct 모듈
struct 모듈은 pack, unpack, calcsize 등의 기능이 있으며, 이것들을 사용하여 정수, 부동 소수점 숫자, 문자열(을 encode 메소드에서 인코딩 한 것)을 bytes 객체로 변환하거나 반대로 bytes 객체에서 이것을 빼낸다.  
  
bytes 객체로 변환하려면 "서식 지정 문자"라는 문자를 조합하여 이것들과 실제로 변환 할 값을 pack 함수에 넘기면 된다. 
서식 지정 문자와 데이터를 unpack 함수에 전달하여 바이너리에서 읽어 낸 bytes 객체에서 필요한 데이터를 꺼낼 수있다.  
  
서식 지정 문자에는 아래과 같은 것이 있다(일부이므로 더 자세한 것은 python 문서를 보기 바란다).  
<pre>
서식 지정    설명	크기(바이트 수)  
c              문자               1  
b              부호있는 정수   1  
B              부호없는 정수   1  
h              부호있는 정수   2  
H              부호없는 정수   2  
i              부호있는 정수   4  
I              부호없는 정수   4  
l              부호있는 정수   4  
L              부호없는 정수   4  
f              부동 소수점       4  
d              부동 소수점       8  
s              문자열             -  
<              리틀 엔디안 지정 -  
>              빅 엔디안 지정  -  
<pre>  
  
예를 들어, 정수를 1 바이트의 부호 있는 정수로 bytes 객체로 변환한다면, 형식 문자열은 *b'를 지정하여 변환하고자 하는 값과 함께 pack 함수에 전달한다.   
```
from struct import pack, unpack, calcsize

data = pack('b', 100)
print(data)
print(f"100 == b'{chr(100)}'")
```
  
문자 ''d''의 ASCII 값은 10 진수 "100"이므로 100을 pack 함수에 전달하여 bytes 객체로 변환하면 "b'd'"가 되는 것에 주의하자. 반대로 얻은 bytes 객체를 정수로 변환하려면 아래처럼 한다.  
```
result = unpack('b', data)
print(result)
```
  
결과를 보면 알 수 있듯이, 얻어진 데이터는 튜플에 포함 되는 것에 주의하자. 여기에서는 형식 문자를 문자 하나 밖에 사용하지 않았기 때문에 얻어진 값도 하나 뿐이지만, 여러 형식 문자열을 지정하면 그 수만큼 데이터가 얻어진다.  
  
  
  
## struct 모듈을 사용하여 바이너리 파일을 읽고 쓰기
struct 모듈을 사용하여 이진 파일에 데이터를 쓰고, 이것을 다시 읽어 보자.  
우선, 여기에서는 튜플을 요소로 하는 리스트를 만들어 둔다.  
```
mydata = [(1, 'FOO', 1023), (2, 'BAR', 80), (3, 'BAZ', 4000)]
```
  
요소가 되는 튜플에는 ID, 이름, 데이터가 담겨 있다. 이 때 이름의 길이가 같게 되어 있는 것에 주의하자. struct 모듈은 (문자 포함)이와 같은 정형 포맷의 데이터를 쉽게 bytes 문자열로 변환 할 수 있다. 여기에서는 이들을 다음과 같은 구성으로 bytes 객체로 대체하기로 한다.  
- ID: 4 바이트의 정수
- 이름: 3 문자열
- 데이터: 4 바이트의 정수
  　
이것들을 서식 지정 문자로 표현하면 "l3sll"가된다.  "s" 앞에 있는 "3"은 이것이 3 문자로 구성된 문자열임을 의미한다.  
  
예를 들어, 목록의 첫 번째 요소를 bytes 객체로 변환한다면 다음과 같다(문자열이 포함 되어 있기 때문에, encode 메소드에서 bytes 타입으로 한번 변환해야 하고, 깔끔하게 쓸 수는 없다).  
```
result = pack('l3sl', mydata[0][0], mydata[0][1].encode(), mydata[0][2])
print(result)
```
3개의 데이터가 하나의 bytes 객체로 팩된다.  
이를테면 이런 식으로 리스트의 각 요소(튜플)를 bytes 객체로 팩하고, 이것을 write 메소드에서 이진 파일에 기록하면 된다는 것이다. 실제 코드는 다음과 같다.  
```
myfile = open('mydata.data', 'wb')

for item in mydata:
    result = pack('l3sl', item[0], item[1].encode(), item[2])
    myfile.write(result)

myfile.close()
```
  
이번에는 쓴 데이터를 읽고, 원래의 데이터로 복원 할 수 있는지 확인 해보자. 이 때 튜플 하나 당 데이터 크기가 필요하다. 이를 확인하 데 사용할 수 있는 것이, calcsize 함수이다. 이것에 서식 지정 문자를 주면 이 데이터의 크기 알 수 있다.  
```
size = calcsize('l3sl')
myfile = open('mydata.data', 'rb')

content = myfile.read(size)
while content:
    restored_data = unpack('l3sl', content)
    print(restored_data)
    content = myfile.read(size)

myfile.close()
```
  
여기에서는 calcsize 함수에서 조사한 데이터 크기마다 read 메소드에서 파일의 내용을 읽고, 이것을 unpack 함수에 전달하여 데이터를 추출하여 화면에 표시하고 있다. 이것이 끝나면 다시 데이터 크기만큼 파일에서 로드하고 파일이 비워 질 때까지 이것을 계속한다.  
  
  
  
## struct 모듈을 사용하여 GIF 파일에서 읽어들이는 코드
필요한 데이터는 다음과 같다.  
- GIF 시그니처는 "GIF" 후에 "87a" 또는 "89a"가 계속
- 그런 다음 2 바이트로 화면의 가로 폭과 세로 폭이 있다
  
이것을 서식 지정 문자로하면 "6shh"가 된다. 이것을 알면 함수는 즉시 쓸 수 있다. 실제 코드는 다음과 같다.  
```
def get_dimension_with_struct(filename):
    myfile = open(filename, 'rb')
    content = myfile.read(10)
    myfile.close()
    (spec, width, height) = unpack('6shh', content)
    print(f'this file is {spec.decode()}, size: {width} x {height}')
```
  
여기에서 첫 번째 10 바이트를 먼저 읽고, 이것을 위의 형식 문자열과 함께 unpack 함수에 전달(바이트 순서를 규정하고 있지 않기 때문에, 환경에 따라서는 잘못된 값으로 변환될도 있다).  
  
unpack 함수는 bytes 오브젝트화 된 "GIF + 사양"과 가로, 세로 폭을 튜플에 모아서 반환 하므로, 3개의 변수 spec, width, height에 대입하도록 한다. 뒤는 이것들을 화면에 표시 할 뿐이다.  
  
struct 모듈을 사용하면 이처럼 비교적 쉽게 정형 데이터를 bytes 객체로 변환하고, 이것을 이진 파일에 기록하거나 바이너리 파일에서 읽을 수 있다. 그러나 특정 포맷을 가진 데이터를 읽고 쓰려면, pickle 모듈 등보다 편리한 모듈도 준비되어 있다.   