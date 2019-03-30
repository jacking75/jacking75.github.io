---
layout: post
title: C# - 정규 표현식 라이브러리 VerbalExpressions
published: true
categories: [.NET]
tags: c# net 정규표현식 VerbalExpressions
---
## 소개
* 기존의 복잡한 방식의 정규식을 이해하기 쉽도로 해주는 라이브러리.
* 처음에는 JavaScript용으로 나왔지만 이후 대부분의 언어를 지원하고 있다.
* 소스 코드는 GitHub에 공개 되어 있다.  
    https://github.com/VerbalExpressions/CSharpVerbalExpressions
* 간단 예제
  
```
[TestMethod]
public void TestingIfWeHaveAValidURL()
{
    // Create an example of how to test for correctly formed URLs
    var verbEx = new VerbalExpressions()
                .StartOfLine()
                .Then( "http" )
                .Maybe( "s" )
                .Then( "://" )
                .Maybe( "www." )
                .AnythingBut( " " )
                .EndOfLine();

    // Create an example URL
    var testMe = "https://www.google.com";

    Assert.IsTrue(verbEx.Test( testMe ), "The URL is incorrect");

    Console.WriteLine("We have a correct URL ");
}
```
  
위의 코드를 기존의 정규식으로 표현하면 아래와 같다.  
```
/^(http)(s)?(\:\/\/)(www\.)?([^\ ]*)$
```
  
* (2013.10.30)Nuget으로 설치한 라이브러리에 실제 소스 코드를 받아와서 빌드한 라이브러리에 있는 기능의 일부가 없음.
  
  
  
## API
### then(value)
* https://github.com/VerbalExpressions/JSVerbalExpressions/wiki/.then(-value-)
* 문자열에서 value와 같은 모든 것을 타겟으로 한다.
  
```
var my_paragraph = "Lorem ipsum. Dolor sit amet. Consectetur adipisicing elit.";
my_paragraph = VerEx().then( "." ).replace( my_paragraph, ". Stop." );
```
  
<pre>
/* Outputs:
Lorem ipsum. Stop. Dolor sit amet. Stop. Consectetur adipisicing elit. Stop.
*/
</pre>
  
  
  
### .endOfLine()
* 줄의 마지막을 가리킨다.
  
```
var lorem = "Lorem ipsum.\nDolor sit amet."
lorem = VerEx().endOfLine().replace( lorem, " Stop." );
```
  
<pre>
/* Outputs:
Lorem ipsum. Stop.
Dolor sit amet. Stop.
*/
</pre>
  
  
  
### .endOfLine( enable )
* .endOfLine 적용 여부
  
```
var lorem = "Lorem ipsum\nLorem ipsum and something else";

var expression = VerEx().endOfLine().then( "Lorem ipsum" );
lorem = lorem.replace( expression , "First replacement" );

expression.endOfLine( false );
lorem = lorem.replace( expression , "Second replacement" );
```
  
<pre>
/* Outputs:
First replacement
Then comes second replacement
*/
</pre>
  
  
  
### .startOfLine()
* 줄의 시작을 가리킨다.
  
```
var lorem = "Lorem ipsum\nDolor sit amet"
lorem = VerEx().startOfLine().replace( lorem, "Newline: " );
```
  
<pre>
/* Outputs:
Newline: Lorem ipsum
Newline: Dolor sit amet
*/
</pre>
  
```
var lineNumber = 1;
var lorem = "Lorem ipsum\nDolor sit amet"
lorem = VerEx().startOfLine().replace( lorem, function() { return lineNumber++ + ". "; } );
```
  
<pre>
/* Outputs:
1. Lorem ipsum
2. Dolor sit amet
*/
</pre>
  
  
```
var lorem = "Lorem ipsum\nAnother Lorem ipsum";
lorem = VerEx().startOfLine().then( "Lorem ipsum" ).replace( lorem, "This gets replaced" );
```
  
<pre>
/* Outputs:
This gets replaced
Another Lorem ipsum
*/
</pre>
  
  
  
### .startOfLine( enable )
  
  
  
### .maybe( value )
* value가 1회 or 0회 발견 되었다면
  
```
var my_paragraph = "Take a look at the link http://google.com and the secured version https://www.google.com";
var expression = VerEx()
                .find( "http" )
                .maybe( "s" )
                .then( "://" )
                .anythingBut( " " );

my_paragraph = expression.replace( my_paragraph, function(link) {
    return "<a href='"+link+"'>"+link+"</a>";
} );
```
  
<pre>
/* Outputs:
Take a look at the link <a href='http://google.com'>http://google.com</a> and the secured version <a href='https://www.google.com'>https://www.google.com</a>
*/
</pre>
  
  
  
### .lineBreak(), .br()
  
```
var lorem = "Lorem ipsum.\r\nDolor sit\namet."
lorem = VerEx().lineBreak().replace( lorem, "<br>\n" );
```
  
<pre>
/* Outputs:
Lorem ipsum.<br>
Dolor sit<br>
amet.
*/
</pre>
  
  
  
### .range()
* Usage: .range( from, to [, from, to ... ] ) Description: Add expression to match a tab character
  
```
VerEx().range( '0', '9', 'a', 'f')
```
  
  
  
## C# 버전
* 내부적으로 C#의 정규표현식 클래스인 RegularExpressions을 사용하므로 이것에 대한 이해도 필요하다.
* RegexOptions 정규식 옵션
    * http://msdn.microsoft.com/ko-kr/library/system.text.regularexpressions.regexoptions.aspx
  
  
  
### BeginCapture
* 하나의 유닛으로 복수의 문자를 다루기 위해 Capture group를 만들기 위해 사용한다.
    * http://neokido.tistory.com/m/post/view/id/307
* ((A)(B(C))) 은 아래의 문자열을 조사한다.  
((A)(B(C)))  
(A)  
(B(C))  
(C)  
  
  
  
### 예제 코드
  
```
VerbalExpressions GetVerbalExpressions_TestMethod1()
{
    var verbEx = VerbalExpressions.DefaultExpression
                .StartOfLine()
                .Then("test").WithOptions(System.Text.RegularExpressions.RegexOptions.IgnoreCase)
                .EndOfLine();

    return verbEx;
}

VerbalExpressions GetVerbalExpressions_TestMethod2()
{
    var verbEx = VerbalExpressions.DefaultExpression
                .StartOfLine()
                .Then("test").WithOptions(System.Text.RegularExpressions.RegexOptions.IgnoreCase)
                ;

    return verbEx;
}

// test 라는 단어 다음에 숫자가 나오는 경우
[TestMethod]
public void TestMethod1()
{
    // test 라는 단어로 조사. test는 대문자 소문자 구분 안한다.
    object[] range = new object[2] { 0, 9 };

    var verbEx = GetVerbalExpressions_TestMethod1();
    var testMe1 = "test0";

    verbEx.Range(range);
    Assert.IsTrue(verbEx.IsMatch(testMe1));


    var verbEx2 = GetVerbalExpressions_TestMethod1();
    var testMe2 = "test9";
    verbEx2.Range(range);
    Assert.IsTrue(verbEx2.IsMatch(testMe2));


    var verbEx3 = GetVerbalExpressions_TestMethod1();
    var testMe3 = "test8";
    verbEx3.Range(range);
    Assert.IsTrue(verbEx3.IsMatch(testMe3));


    var verbEx4 = GetVerbalExpressions_TestMethod1();
    var testMe4 = "tesst";
    verbEx4.Range(range);
    Assert.IsFalse(verbEx4.IsMatch(testMe4));


    var verbEx5 = GetVerbalExpressions_TestMethod1();
    var testMe5 = "testt0";
    verbEx5.Range(range);
    Assert.IsFalse(verbEx4.IsMatch(testMe5));
}

// test의 t 다음에 공백
[TestMethod]
public void TestMethod2()
{
    // test 라는 단어로 조사. test는 대문자 소문자 구분 안한다.
    var verbEx1 = GetVerbalExpressions_TestMethod2();
    verbEx1.Add(" ");
    var testMe1 = "test ";

    Assert.IsTrue(verbEx1.Test(testMe1));

    var verbEx2 = GetVerbalExpressions_TestMethod2();
    verbEx2.Add(" ");
    var testMe2 = "test fdf";

    Assert.IsTrue(verbEx2.Test(testMe2));

    var verbEx3 = GetVerbalExpressions_TestMethod2();
    verbEx3.Add(" ");
    var testMe3 = "test2";

    Assert.IsFalse(verbEx3.Test(testMe3));
}

// test의 t 다음에 영어 이외의 문자 시작 조사
[TestMethod]
public void TestMethod3()
{
    // test 라는 단어로 조사. test는 대문자 소문자 구분 안한다.

    object[] range = new object[] { 'a', 'z', 'A', 'Z' };

    var verbEx = GetVerbalExpressions_TestMethod1();
    var testMe1 = "test_";

    verbEx.AnythingBut(range.ToString());
    Assert.IsTrue(verbEx.IsMatch(testMe1));

    var verbEx2 = GetVerbalExpressions_TestMethod1();
    var testMe2 = "testt";

    verbEx2.AnythingBut(range.ToString());
    Assert.IsFalse(verbEx2.IsMatch(testMe2));

    var verbEx3 = GetVerbalExpressions_TestMethod1();
    var testMe3 = "test2";

    verbEx3.AnythingBut(range.ToString());
    Assert.IsTrue(verbEx3.IsMatch(testMe3));
}

// 한글 포함 여부 체크
[TestMethod]
public void TestMethod4()
{
    object[] range = new object[] { 'ㄱ', 'ㅎ', '가', '힣' };

    var verbEx = GetVerbalExpressions_TestMethod1();
    var testMe1 = "test아";

    verbEx.Range(range);
    Assert.IsTrue(verbEx.IsMatch(testMe1));

    var verbEx2 = GetVerbalExpressions_TestMethod1();
    var testMe2 = "test2T_";

    verbEx2.Range(range);
    Assert.IsFalse(verbEx2.IsMatch(testMe2));
}
```
  
* url 주소 형식이 맞는지 조사
  
```
var verbEx = VerbalExpressions.DefaultExpression
            .StartOfLine()
            .Then("http")
            .Maybe("s")
            .Then("://")
            .Maybe("www.")
            .AnythingBut(" ")
            .EndOfLine();

var testMe = "https://www.google.com";

Assert.IsTrue(verbEx.Test(testMe));
```
  
* LineBreak가 포함되었는지 조사
  
```
//Arrange
var verbEx = VerbalExpressions.DefaultExpression;
string text = string.Format("testin with {0} line break", Environment.NewLine);

//Act
verbEx.LineBreak();
//Assert
Assert.IsTrue(verbEx.Test(text));
```
  
* tab이 포함되었는지 조사
  
```
//Arrange
var verbEx = VerbalExpressions.DefaultExpression;
string text = string.Format("text that contains {0} a tab", @"\t");

//Act
verbEx.Tab();

//Assert
Assert.IsTrue(verbEx.Test(text));
```
  
* 단어 개수 얻기
  
```
//Arrange
var verbEx = VerbalExpressions.DefaultExpression;
string text = "three words here";
int expectedCount = 3;

//Act
verbEx.Word();
Regex currentExpression = verbEx.ToRegex();
int result = currentExpression.Matches(text).Count;

//Assert
Assert.AreEqual(expectedCount, result);
```
  
* 이메일 형식이 맞는지 조사
  
```
public static readonly CommonRegex Url = new CommonRegex(1, @"((([A-Za-z]{3,9}:(?:\/\/)?)(?:[^-;:&=\+\$,\w]+@)?[A-Za-z0-9.-]+(:[0-9]+)?|(?:www.|[^-;:&=\+\$,\w]+@)[A-Za-z0-9.-]+)((?:\/[\+~%\/.\w_]*)?\??(?:[-\+=&;%@.\w-_]*)#?‌​(?:[\w]*))?)");
public static readonly CommonRegex Email = new CommonRegex(2, @"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}");
        
var verbEx = VerbalExpressions.DefaultExpression;
verbEx.StartOfLine().Then(CommonRegex.Email);

var isMatch = verbEx.IsMatch("test@github.com");
Assert.IsTrue(isMatch, "Should match email address");
```
  
* url 형식이 맞는지 조사
  
```
public static readonly CommonRegex Url = new CommonRegex(1, @"((([A-Za-z]{3,9}:(?:\/\/)?)(?:[^-;:&=\+\$,\w]+@)?[A-Za-z0-9.-]+(:[0-9]+)?|(?:www.|[^-;:&=\+\$,\w]+@)[A-Za-z0-9.-]+)((?:\/[\+~%\/.\w_]*)?\??(?:[-\+=&;%@.\w-_]*)#?‌​(?:[\w]*))?)");
public static readonly CommonRegex Email = new CommonRegex(2, @"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}");
        
var verbEx = VerbalExpressions.DefaultExpression;
verbEx.StartOfLine()
      .Then(CommonRegex.Url);

Assert.IsTrue(verbEx.IsMatch("http://www.google.com"), "Should match url address");
Assert.IsTrue(verbEx.IsMatch("https://www.google.com"), "Should match url address");
Assert.IsTrue(verbEx.IsMatch("http://google.com"), "Should match url address");
```
  
* 'test'와 'ing'가 연달아서 있는지 조사(Add를 사용)
  
```
var verbEx = VerbalExpressions.DefaultExpression;
verbEx.Add("test")
    .Add("ing")
    .StartOfLine();

string text = "testing1234";
Assert.IsTrue(verbEx.IsMatch(text), "Should match that the text starts with test");


var verbEx2 = VerbalExpressions.DefaultExpression;
verbEx2.Add("test")
    .Add("ing")
    .StartOfLine();

string text2 = "test123ing";
Assert.IsFalse(verbEx2.IsMatch(text2), "Should match that the text starts with test");
```
  
* 대소문자 구별 여부(WithAbyCase)
  
```
var verbEx = VerbalExpressions.DefaultExpression;
verbEx.Add("www")
    .WithAnyCase();

var isMatch = verbEx.IsMatch("wWw");
Assert.IsTrue(isMatch, "Should match any case");

var verbEx = VerbalExpressions.DefaultExpression;
verbEx.Add("www")
    .WithAnyCase(false);

var isMatch = verbEx.IsMatch("wWw");
Assert.IsFalse(isMatch, "Should not match any case");
```
  
* 대문자, 소문자 구별 안하는 옵션 설정(AddModifier)
  
```
VerbalExpressions verbEx = VerbalExpressions.DefaultExpression;
verbEx.Add( "teststring" ).AddModifier( 'i' );

Assert.IsTrue( verbEx.IsMatch( "TESTSTRING" ) );
```
  
* Capture grouping
  
```
// Arrange
VerbalExpressions verbEx = VerbalExpressions.DefaultExpression;

// Act
verbEx.BeginCapture()
      .Add( "com" )
      .Or( "org" )
      .EndCapture();

// Assert
Assert.AreEqual( "((com)|(org))", verbEx.ToString() );
```
  
* 특정 단어로 끝나는지 조사(EndOfLine)
  
```
var verbEx = VerbalExpressions.DefaultExpression;
verbEx.Add(".com").EndOfLine();

var isMatch = verbEx.IsMatch("www.google.com");
Assert.IsTrue(isMatch, "Should match '.com' in end");


var verbEx = VerbalExpressions.DefaultExpression;
verbEx.Add(".com").EndOfLine();

var isMatch = verbEx.IsMatch("http://www.google.com/");
Assert.IsFalse(isMatch, "Should not match '/' in end");
```
  
* Multiple
  
```
 //Arrange
var verbEx = VerbalExpressions.DefaultExpression;
string text = "testesting 123 yahoahoahou another test";
string expectedExpression = "y(aho)+u";
//Act
verbEx.Add("y")
    .Multiple("aho")
    .Add("u");

//Assert
Assert.IsTrue(verbEx.Test(text));
Assert.AreEqual(expectedExpression, verbEx.ToString());
```
  
* or 기법 사용
  
```
public void Or_AddComOrOrg_DoesMatchComAndOrg()
{
    var verbEx = VerbalExpressions.DefaultExpression;
    verbEx.Add("com").Or("org");

    Console.WriteLine(verbEx);
    Assert.IsTrue(verbEx.IsMatch("org"), "Should match 'org'");
    Assert.IsTrue(verbEx.IsMatch("com"), "Should match 'com'");
}

[Test]
public void Or_AddComOrOrg_RegexIsAsExpecteds()
{
    var verbEx = VerbalExpressions.DefaultExpression;
    verbEx.Add("com").Or("org");

    Assert.AreEqual("(com)|(org)", verbEx.ToString());
}
```
  
* 범위로 검색(한글, 영어, 숫자, 한문 검색할 때 유용)
  
```
[Test]
public void Range_WhenOddNumberOfItemsInArray_ShouldAppendLastElementWithOrClause()
{
    //Arrange
    var verbEx = VerbalExpressions.DefaultExpression;
    string text = "abcd7sdadqascdaswde";
    object[] range = new object[3] { 1, 6, 7 };

    //Act
    verbEx.Range(range);
    //Assert
    Assert.IsTrue(verbEx.IsMatch(text));
}

[Test]
public void Range_WhenOddNumberOfItemsInArray_ShouldAppendWithPipe()
{
    //Arrange
    var verbEx = VerbalExpressions.DefaultExpression;
    object[] range = new object[3] { 1, 6, 7 };
    string expectedExpression = "[1-6]|7";

    //Act
    verbEx.Range(range);

    //Assert
    Assert.AreEqual(expectedExpression, verbEx.ToString());
}
```
  
  
  
## 도움말
* JS 버전
    * https://github.com/VerbalExpressions/JSVerbalExpressions/wiki
* Go 버전
    * http://godoc.org/github.com/VerbalExpressions/GoVerbalExpressions
* 정규 표현식 표
    * http://j07051.tistory.com/554
* 정규 표현식 강좌
    * http://blog.eairship.kr/category/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%20%EA%B4%80%EB%A0%A8/%EC%A0%95%EA%B7%9C%20%ED%91%9C%ED%98%84%EC%8B%9D
  
