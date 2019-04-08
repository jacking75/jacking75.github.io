---
layout: post
title: C# - 정규 표현식
published: true
categories: [.NET]
tags: c# .net 정규표현식
---
## 문자 집합
- [] 안에 일치시키려는 문자를 일일이 나열해주거나 범위 지정이 가능할 경우 그 범위만큼만 지정해 주면 된다.
- [a-z]   : 영어 소문자
- [A-Z]  : 영어 대문자
- [0-9]   : 숫자
- [aeiou]: 모음
- [k-p] : 알파벳에서 k 부터 p 까지만.
- [4-8] : 숫자 4에서 8까지만.
- m[aeiou]n 이라고 정규식 패턴을 쓰면 man, mon, min, men, min, mun  식의 단어를 찾아준다. 1[4-7]8 이라고 하면 148, 158, 168, 178 을 찾아주는 식이다.
- 문자집합에서 [a-z] 나 [5-8]처럼 범위를 지정하는 경우는 상식적으로 순서가 매겨질때 가능하다
  
  
  
## 한글 찾기
- [가-힣] : 한글 한글자면 뭐든지 okay.
    - 그래서 최[가-힣]규이라고 하면 최완규도 되고 최상규도 되고 최원규도 되고... 최홍규까지 된다. 최와 규자 사이에 어떤 글자가 와도 상관없다는 말이다. 물론, 우리 삼형제 처렴 규자 돌림에 완-상자까지만 있다면 최[완원상]규 라고 하거나 최[사-자]규 처럼 완원상이 포함될 범위를 지정해 주면 된다.
- .*[가-힣]+.* : 앞에 주절주절 영어든 숫자든 한글이든 마구 반복돼도 상관 없고 가운데 한글이 한 글자 이상 반복된 다음 또 무슨 문자가 반복돼도 상관없다.
    - .* 는 어떤 한 문자가 0개 이상 반복돼도 상관없다는 말이라고 했다. 따라서 그 가운데 [가-힣]+ 라는 것만 신경쓰면 된다. 이 패턴은 한글 한 글자에 해당한다.
  
```
// Java
String strText = "aaa한글 테스트aaa";
if(strText.matches(".*[ㄱ-ㅎㅏ-ㅣ가-힣]+.*")) {
    System.out.println("한글포함");
}
```
  
- 한글 최소 2~5글자 입력가능
  
```
[가-힣]{2,5}
```
  
  
  
## 영문 자막/한글 자막 두줄 찾기 패턴
- (^[ a-zA-Z0-9,.!?']+$)\n(.*[가-힣]+.*)
  
  
## 4.6%를 찾는다
  
```
[0-9]{1,3}\.?[0-9]*%
```
  
  
## 전화번호를 찾는다.
  
```
\d{2,3}-\d{3,4}-\d{4}
[0-9]{2,3}-[0-9]{3,4}-[0-9]{4}
```
  
  
## Url 주소를 가져온다.
  
```
http://([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?
```
  
  
## 나이(textBox)
  
```
[0-9]{1,2}
1?[0-9]?[0-9]
```
  
  
## 주민등록번호
  
```
[0-9]{6}-[0-9]{7}
[0-9][0-9][01][0-9][0123][0-9]-[1234][0-9]{6}
```
  
  
## 특수 문자 제거
  
```
// C#
string input = @"특수문자 !@#$%^& 제거 합시다~~...!!!";
input = Regex.Replace(input, @"[^a-zA-Z0-9가-힣]", "", RegexOptions.Singleline);
Debug.WriteLine(input);
```
  
  
## 대-소문자 알파벳, 숫자, 한글만 있는 문자열인지 조사
  
```
// C#
string pattern = @"^[a-zA-Z0-9가-힣]*$";
Regex.IsMatch(strText, pattern);
```
  
<pre>
< 설명 >
^
입력의 시작 위치를 의미한다. 여러 줄 모드에서는 줄바꿈 뒤도 의미한다
/^A/
"An A"에서 시작 위치 바로 뒤의 A는 일치하지만 마지막의 A는 일치하지 않는다.
^[a-z]
 첫 글자는 반드시 소문자 a-z 사이의 문자를 뜻한다

$
입력의 마지막 위치를 의미한다. 여러 줄 모드에서는 줄바꿈 앞도 의미한다.
/a$/
"Cocoa"에서 마지막에 있는 'a'와 일치한다.
[a-z]$
마지막 글자는 반드시 소문자 a-z사이의 문자를 뜻한다

*
* 앞의 문자가 0번 이상 반복됨을 의미한다.

선택 기호: "|" 기호는 여러 식 중에서 하나를 선택한다.
 "abc|adc"는 abc라는 문자열과 adc라는 문자열을 모두 포함한다.
묶기 기호: (와 )로 여러 식을 하나로 묶을 수 있다.
 "abc|adc"와 "a(b|d)c"는 같은 의미를 가진다.

[] : [과 ] 사이의 문자 중 하나를 선택한다. "|"를 여러 개 쓴 것과 같은 의미를 가진다.
[abc]d는 ad, bd, cd를 뜻한다. 또한, "-" 기호와 함께 쓰면 문자의 범위를 지정할 수 있다.
[a-z]는 a부터 z까지 중 하나, [1-9]는 1부터 9까지 중의 하나를 뜻한다.
[^] : [^과 ]  사이의 문자를 제외한 나머지 하나를 선택한다.
[^abc]d는 ad, bd, cd는 포함하지 않고 ed, fd 등을 포함한다.
[^a-z]는 알파벳 소문자로 시작하지 않는 모든 문자를 나타낸다.
</pre>
  
  
## 문자열이 알파벳으로만 구성되어 있는지를 확인하는 함수
  
```
public bool IsAlpabet(string strValue)
{
   if (strValue == null || strValue.Length < 1)
      return false;

   Regex reg = new Regex(@"^[a-zA-Z]+$");
   return reg.IsMatch(strValue);
}
```
  
  
## 시작문자열이 알파벳이고, 알파벳과 숫자로 이루어진 문자열인지 여부 확인하는 함수
  
```
public bool IsAlpaNumber(string strValue)
{
   if (strValue == null || strValue.Length < 1)
      return false;

   Regex reg = new Regex(@"^[a-zA-Z]+[0-9]*$");
   return reg.IsMatch(strValue);
}
```
  
  
## 숫자, 영문자, 한글, 숫자+영문자, 숫자+영문자+한자, 숫자+영문자+한자+한글
  
```
/// <summary>
/// 정규식 조건에 맞는 문자열인지 체크합니다.
/// </summary>
/// <param name="type">1 : 숫자, 2 : 영문자, 3 : 한글, 4 : 숫자+영문자,
///                             5 : 숫자+영문자+한자, 6 : 숫자+영문자+한자+한글</param>
/// <param name="plainText">문자열</param>
/// <returns></returns>
public static bool IsRegexMatch(int type, string plainText)
{
           Regex rx;

           switch (type)
           {
                     case 1 :
                                rx = new Regex(@"^[0-9]*$", RegexOptions.None);
                                break;
                     case 2 :
                                rx = new Regex(@"^[a-zA-Z]*$", RegexOptions.None);
                                break;
                     case 3:
                                rx = new Regex(@"^[가-힣]*$", RegexOptions.None);
                                break;
                     case 4:
                                rx = new Regex(@"^[a-zA-Z0-9]*$", RegexOptions.None);
                                break;
                     case 5:
                                rx = new Regex(@"^[a-zA-Z0-9一-龥]*$", RegexOptions.None);
                                break;
                     case 6:
                                rx = new Regex(@"^[a-zA-Z0-9가-힣一-龥]*$", RegexOptions.None);
                                break;
                     default :
                                return false;
           }

           return (string.IsNullOrEmpty(plainText)) ? false : rx.IsMatch(plainText);
}
```
  
  
  
## 참고
- 정규식 소개 [http://www.nextree.co.kr/p4327/]
- 正規表現入門　星の高さを求めて [http://www.slideshare.net/sinya8282/ss-32629428]
- 정규표현식(Regular Expression) [http://www.hoons.net/Lecture/View/635]
- 정규 표현식(Regular expression) [http://j07051.tistory.com/554]
- C# 정규 표현식에 대해서 알아보자  [http://durubiz.tistory.com/entry/C-%EC%97%90%EC%84%9C-%EC%A0%95%EA%B7%9C%ED%91%9C%ED%98%84%EC%8B%9D-RegexIsMatch-%EC%9D%B4%EC%9A%A9]
- [連載:正規表現] Unicode文字プロパティについて(1)
    - http://techracho.bpsinc.jp/hachi8833/2013_09_13/13433
    - http://techracho.bpsinc.jp/hachi8833/2013_09_27/13713
    - http://techracho.bpsinc.jp/hachi8833/2013_10_15/13889
    - http://techracho.bpsinc.jp/hachi8833/2014_01_28/15246
