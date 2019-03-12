---
layout: post
title: C# - Unit Test Mock
published: true
categories: [.NET]
tags: c# .net unittest mock
---
## 왜 사용하는가?
- Mock 라이브러리는 단위 테스트를 쉽게 하기 위해 존재합니다.
- 예를 들어서, 한 클래스 테스트에 데이터베이스를 활용해야 하는 경우가 있다고 가정을 합시다. 이 경우 직접 데이터베이스에 정보를 넣었다 뺐다 하면 상당히 큰 부담이 됩니다.
- 이런 부담을 주지 않고, 데이터베이스의 행동을 흉내냄으로써 테스트를 더욱 용이하게 합니다.
- 하지만 이런 용이함을 얻기 위해서는 인터페이스들에 의해서만 프로그램을 조작하는 설계를 해야 하기는 합니다. (이런 설계가 처음에는 많이 어색하지만, 나중에는 이런 설계가 더 나은 설계임을 깨닫게 됩니다.)
- 간단히 이야기해서 Mock 라이브러리는 복잡하고 시간이 걸리는 테스트의 시간을 줄여주기 위해서 존재한다고 볼 수 있습니다.
  
  
## 설치
- Nuget
  
  
## 예제
  
```
public class Book
{
	public string Name { get; set; }
	public int Price { get; set; }
	public string ISBN { get; set; }
}

public interface IBookRepository
{
	List<Book> GetBooks();
}

public class PriceProcessor
{
	public PriceProcessor(IBookRepository bookRepository)
	{
		bookRepo = bookRepository;
	}

	public int GetTotalPrice()
	{
		int result = 0;

		foreach (Book b in bookRepo.GetBooks())
		{
			result += b.Price;
		}

		return result;
	}

	private IBookRepository bookRepo;
}

[TestFixture]
public class PriceProcessorTests
{
	[Test]
	public void GetTotalPrice_NoDiscount_ReturnCorrectPriceSum()
	{
		List<Book> bookList = new List<Book>()
		{
			new Book() {ISBN="1234", Price=10000, Name="C언어 입문"},
			new Book() {ISBN="1235", Price=20000, Name="C++ 입문"},
			new Book() {ISBN="1236", Price=30000, Name="ASP.NET MVC3 완성"},
		};

		Mock<IBookRepository> bookRepoMock = new Mock<IBookRepository>();
		bookRepoMock.Setup(b => b.GetBooks()).Returns(bookList);

		PriceProcessor priceProcessor = new PriceProcessor(bookRepoMock.Object);

		Assert.That(priceProcessor.GetTotalPrice(), Is.EqualTo(60000));
	}
}
```
  
- GetBooks함수가 불리면, bookList를 반환하라고 명령을 내리고 있습니다. 그리고 완성이 된 bookRepoMock을 Object 속성을 이용해서 priceProcessor에 넘겨 주고 있습니다
  
  
  
## 함수가 불려졌는지 확인
  
```
public class BestSellers
{
	public BestSellers(IBookRepository bookRepository)
	{
		bookRepo = bookRepository;
	}

	public Result AddBook(Book book)
	{
		if (book.Name.Contains("<") || book.Name.Contains(">"))
		{
			return Result.XssAlert;
		}

		bookRepo.AddBook(book);
		return Result.Success;
	}

	IBookRepository bookRepo;
}

public interface IBookRepository
{
	List<Book> GetBooks();
	bool AddBook(Book book);
}

[Test]
public void AddBook_ContainsTag_IBookRepositoryAddBookIsNotCalled()
{
	Book b = new Book() { Name="<iframe></iframe>"};

	Mock<IBookRepository> mock = new Mock<IBookRepository>();

	mock.Setup(r => r.AddBook(b)).Verifiable();

	BestSellers bestSellers = new BestSellers(mock.Object);
	bestSellers.AddBook(b);

	mock.Verify(r => r.AddBook(b), Times.Never());

}
```
  
<img src="http://cfile1.uf.tistory.com/image/2711653D5280A22F32C975">  
  
  
  
## Mock 객체에서 생성된 함수들을 매개 변수에 따라 동작을 조절
  
```
public class Book
{
	public string Name { get; set; }
	public int Price { get; set; }
	public string ISBN { get; set; }
}

public void ReducePrice(float reductionRatio)
{
	foreach (Book b in bookRepo.GetBooks())
	{
		b.Price = (int)(reductionRatio * b.Price);
		bookRepo.UpdateBook(b);
	}
}

[Test]
public void ReducePrice_ValueBetween0And1_IBookRepositoryUpdateBookCalled()
{
	List<Book> bookList = new List<Book>()
	{
		new Book() {ISBN="1234", Price=10000, Name="C언어 입문"},
		new Book() {ISBN="1235", Price=20000, Name="C++ 입문"},
		new Book() {ISBN="1236", Price=30000, Name="ASP.NET MVC3 완성"},
	};

	Mock<IBookRepository> bookRepoMock = new Mock<IBookRepository>();
	bookRepoMock.Setup(b => b.GetBooks()).Returns(bookList);

	PriceProcessor priceProcessor = new PriceProcessor(bookRepoMock.Object);
	priceProcessor.ReducePrice(0.1f);

	bookRepoMock.Verify(r => r.UpdateBook(It.IsAny<Book>()),

	Times.Exactly(bookList.Count));
}
```
  
<img src="http://cfile9.uf.tistory.com/image/272FA5455280A22F07C545">  
  
```
public Result AddBook(Book book)
{
	if (book.Name.Contains("<") || book.Name.Contains(">"))
	{
		return Result.XssAlert;
	}

	bool added = bookRepo.AddBook(book);

	if (added == false)
	{
		return Result.AlreadyExist;
	}

	return Result.Success;

}

[Test]
public void AddBook_AddSameBookAgain_ReturnsAlreadyExist()
{

	Book b = new Book() { ISBN = "1234", Name = "abc" };

	int count = 0;

	Mock<IBookRepository> mock = new Mock<IBookRepository>();

	mock.Setup(r => r.AddBook(b)).Callback(() => count++).Returns(() => count <= 1);

	BestSellers bestSellers = new BestSellers(mock.Object);

	Result result1 = bestSellers.AddBook(b);

	Result result2 = bestSellers.AddBook(b);


	Assert.That(result1, Is.EqualTo(Result.Success));
	Assert.That(result2, Is.EqualTo(Result.AlreadyExist));
}
```
  
<img src="http://cfile27.uf.tistory.com/image/2553073F5280A23024C7DD">  
  
  
  
## 참고
- http://blog.naver.com/empty_wagon/20151928869
- http://blog.naver.com/empty_wagon/20151986599
- [Moq을 활용하여 .NET으로 목을 만들어 테스트 하자](http://news.mynavi.jp/articles/2009/06/15/moq/index.html)
- [Moq ＆ Fakes Framework을 사용한 실험적 유닛테스터](http://www.buildinsider.net/hub/bioff/o3)