---
layout: post
title: C# - Unit Test
published: true
categories: [.NET]
tags: c# net unittest 유닛테스트
---
### 준비
- 보통 프로그램이나 라이브러리 프로젝트에 추가 프로젝트로 유닛테스트 프레임워크를 사용하는 프로젝트를 만든다.
- 이렇게 하면 유닛테스트에 영향을 받지 않으면서 기존처럼 개발을 할 수 있다.  
<img src="http://cfile24.uf.tistory.com/image/260E513B5280A06A09B75D" width="400px">  
  
- 유닛테스트 프로젝트에 테스트 메소드를 만들면 테스트 탐색기에 리스트로 나온다.  
<img src="http://cfile2.uf.tistory.com/image/2577BB3D5280A06B374B53" width="600px">
  
  
  
### 유닛테스트 초기화와 실행 순서

<img src="http://cfile7.uf.tistory.com/image/2346933C5280A06B182D70" width="600px">
  
  
  
## 지정한 유닛테스트만 실행
- 유닛테스트 리스트에서 특정 테스트만 실행하고 싶은 경우 그 항목만 선택 후 테스트를 실행한다  
<img src="http://cfile26.uf.tistory.com/image/2777BC3D5280A06B38E4BF" width="600px">
  
  
  
### 카테고리
- 유닛테스트를 비슷한 항목끼리 카테고리로 묶어서 카테고리 별로 테스트 할 수 있다.  
<img src="http://cfile2.uf.tistory.com/image/263E57355280A06C29E472" width="600px">
  
  
### 유닛 테스트 예제
  
```
public void Test주고_받는_점수_계산하기()
{
	PrivateType TestObject = new PrivateType(typeof(YakuLibNET.ScoreCalculator));

	bool Is부모가_이긴 = true;
	int 이긴_플레이어_점수 = 0;

	object Result;
	YakuLibNET.KUK_END_SCORE ResultScore; 


	// [테스트-부모가 이긴 경우]
	Is부모가_이긴 = true;
	이긴_플레이어_점수 = 3900;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(3900, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(1300, ResultScore.부모윈_진_자식이_내는_점수);


	Is부모가_이긴 = true;
	이긴_플레이어_점수 = 2000;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(이긴_플레이어_점수, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(700, ResultScore.부모윈_진_자식이_내는_점수);


	Is부모가_이긴 = true;
	이긴_플레이어_점수 = 7700;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(이긴_플레이어_점수, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(2600, ResultScore.부모윈_진_자식이_내는_점수);


	Is부모가_이긴 = true;
	이긴_플레이어_점수 = 11600;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(이긴_플레이어_점수, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(3900, ResultScore.부모윈_진_자식이_내는_점수);


	Is부모가_이긴 = true;
	이긴_플레이어_점수 = 10600;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(이긴_플레이어_점수, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(3600, ResultScore.부모윈_진_자식이_내는_점수);


	// [자신이 이긴 경우]
	Is부모가_이긴 = false;
	이긴_플레이어_점수 = 1300;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(이긴_플레이어_점수, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(700, ResultScore.자식윈_진_부모가_내는_점수);
	Assert.AreEqual(400, ResultScore.자식윈_진_자식이_내는_점수);


	Is부모가_이긴 = false;
	이긴_플레이어_점수 = 5200;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(이긴_플레이어_점수, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(2600, ResultScore.자식윈_진_부모가_내는_점수);
	Assert.AreEqual(1300, ResultScore.자식윈_진_자식이_내는_점수);


	Is부모가_이긴 = false;
	이긴_플레이어_점수 = 7100;

	Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);
	ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(이긴_플레이어_점수, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(3600, ResultScore.자식윈_진_부모가_내는_점수);
	Assert.AreEqual(1800, ResultScore.자식윈_진_자식이_내는_점수);
}
```
  
  
  
### 테스트 함수 리스트
  
<img src="http://cfile23.uf.tistory.com/image/23436B405280A06C123E7A">
  
  
  
### public이 아닌 메소드 테스트 하기
- 유닛테스트를 할 때 가장 골치 아픈 것 중의 하나가 테스트할 함수가 public가 아닌 경우이다. 다행히 닷넷에서는 테스트 할 때만 public로 변경해 주는 기능이 있다(기존 코드 수정 없이).
- 정적(static) 메소드는 PrivateType 사용
    - http://msdn.microsoft.com/ko-kr/library/microsoft.visualstudio.testtools.unittesting.privatetype.aspx
- 일반 메소드는 PrivateObject 사용
    - http://msdn.microsoft.com/ko-kr/library/microsoft.visualstudio.testtools.unittesting.privateobject.aspx
  
```
public void Test주고_받는_점수_계산하기()
{
	 // ScoreCalculator는 private 멤버 함수이다.
	PrivateType TestObject = new PrivateType(typeof(YakuLibNET.ScoreCalculator));

	// [테스트-부모가 이긴 경우]
	bool Is부모가_이긴 = true;
	int 이긴_플레이어_점수 = 3900;

	object Result = TestObject.InvokeStatic("주고_받는_점수_계산하기", Is부모가_이긴, 이긴_플레이어_점수);

	YakuLibNET.KUK_END_SCORE ResultScore = (YakuLibNET.KUK_END_SCORE)Result;

	Assert.AreEqual(3900, ResultScore.이긴_플레이어가_받는_점수);
	Assert.AreEqual(1300, ResultScore.부모윈_진_자식이_내는_점수);
}

public class PrivateAccess
{
	private static string NameStatic
	{
		get { return typeof(PrivateAccess).Name; }
	}

	private string Name { get; set; }

	public PrivateAccess(string name)
	{
		this.Name = name;
	}

	private int Add(int lhs, int rhs)
	{
		return lhs + rhs;
	}

	private static int AddStatic(int lhs, int rhs)
	{
		return lhs + rhs;
	}

	private int Subtract(int lhs, int rhs)
	{
		return lhs - rhs;
	}
}

[TestMethod()]
public void PrivateAccessConstructorTest()
{
	// Arange
	PrivateObject po = new PrivateObject(typeof(PrivateAccess), new object[] { "안녕" });
	string name = "안녕";

	// Act
	// PrivateAccess target = new PrivateAccess(name);

	// Assert
	Assert.AreEqual(name, po.GetProperty("Name"));  
}

public void AddTest()
{
	PrivateObject po = new PrivateObject(typeof(PrivateAccess), new object[] { "안녕" });

	//PrivateAccess_Accessor target = new PrivateAccess_Accessor(param0);
	int lhs = 2;
	int rhs = 3;
	int expected = 5;
	int actual;

	actual = (int) po.Invoke("Add", new object[]{lhs, rhs});

	Assert.AreEqual(expected, actual);
}

public void AddStaticTest()
{
	PrivateType pt = new PrivateType(typeof(PrivateAccess));

	int lhs = 1;
	int rhs = 2;
	int expected = 3;
	int actual;

	actual = (int) pt.InvokeStatic("AddStatic", new object[] { lhs, rhs });

	Assert.AreEqual(expected, actual);
}

[TestMethod()]
[DeploymentItem("SampleLibrary.dll")]
public void NameTest()
{
	PrivateObject po = new PrivateObject(new PrivateAccess("hello"));

	// PrivateAccess_Accessor target = new PrivateAccess_Accessor(param0);

	string expected = "hello";
	string actual = po.GetProperty("Name") as string;

	Assert.AreEqual(expected, actual);
}

[TestMethod()]
public void NameStaticTest()
{
	PrivateType pt = new PrivateType(typeof(PrivateAccess));

	string actual = pt.GetStaticProperty("NameStatic") as string;

	Assert.AreEqual("PrivateAccess", actual);
}
```
  
  
  
### public이 아닌 멤버나 프로퍼티 값 설정 하기
  
```
calss TEST
{
	public int N1 { get; private set; }
}

var po = new PrivateObject(typeof(TEST));
po.SetFieldOrProperty("N1", 1);

var test = (TEST)po.Target;
```
  
  
  
### static 클래스의 private 필드 설정하기
  
```
public class UniqueNumberRepository
{
	const int MaxStockCount = 200000;
	const int MinStockCount = 50000;
	static Int64 seqNumber = 0;
	static Int64 lastNumber = 0;
}

PrivateType TestObject = new PrivateType(typeof(UniqueNumberRepository));
TestObject.SetStaticField("seqNumber", 1);
TestObject.SetStaticField("lastNumber", 10);
```
  
  
  
### static 클래스의 private 필드 참조하기
- [PrivateType.GetStaticField 메서드 (String)](http://msdn.microsoft.com/ko-kr/library/ms244060.aspx)
  
  
  
### System.Transactions를 사용한 DB 유닛테스트
- 다중 커밋을 지원.
- 유닛테스트 코드에서 트랜잭션을 시작하여 제품 코드를 호출하고, 제품 코드측에서 데이터베이스에 변경을 한 후 커밋하여도 그 뒤의 유닛테스트 측에서 롤백하여 데이터베이스에 변경을 남기지 않는다.  
<img src="http://cfile21.uf.tistory.com/image/2133EE3C5280A06D254488">
  
  
  
### 예외 발생 여부 테스트
  
```
// InvalidOperationException 이 발생하면 테스트 성공
[TestMethod()]
[ExpectedException(typeof(InvalidOperationException))]
public void TestMethod() 
{
 // 여기서 InvalidOperationException 예외가 발생하는지 테스트
}
```
  
  
  
### 참고
- [(일어)Visual Studio 유닛테스트 기능대전](http://codezine.jp/article/corner/355)  