---
layout: post
title: C# - List, Tuple, Queue, SortedList, SortedDictionary, SortedSet
published: true
categories: [.NET]
tags: c# net Tuple Queue SortedList SortedDictionary SortedSet
---
### List
#### 정렬
- https://msdn.microsoft.com/ja-jp/library/w56d4y5z(v=vs.110).aspx
- https://msdn.microsoft.com/ja-jp/library/b0zbh7b6(v=vs.110).aspx
- https://msdn.microsoft.com/ja-jp/library/234b841s(v=vs.110).aspx
  
```
List<UserWeekTotalNetProfitResult> TopPlyser = new List<UserWeekTotalNetProfitResult>();

TopPlyser.Sort(PlayerCompare);

static int PlayerCompare(UserWeekTotalNetProfitResult user1, UserWeekTotalNetProfitResult user2)
{
	if (user1.TotalScore == user2.TotalScore)
	{
		if (user1.Level == user2.Level)
		{
			// UID는 절대 동점이 없음
			if (user1.UID > user2.UID)
			{
				return -1;
			}
			else
			{
				return 1;
			}
		}
		else
		{
			if (user1.Level > user2.Level)
			{
				return -1;
			}
			else
			{
				return 1;
			}
		}
	}
	else
	{
		if (user1.TotalScore > user2.TotalScore)
		{
			return -1;
		}
		else
		{
			return 1;
		}
	}
}
```
  
  
#### Find
숫자뿐만이 아닌 문자 검색도 기본적으로 가능하다.
  
  
#### IndexOf()
지정된 개체를 검색하고, 전체 List<(Of <(T>)>)에서 처음으로 검색한 개체의 인덱스(0부터 시작)를 반환합니다.
  
  
#### 리스트의 모든 내용을 배열로 복사
  
```
string[] TemStrs = new string[TypeName.Count];
TypeName.CopyTo(TemStrs);
```
  
  
#### 리스트 간에 복사
  
```
list<string> list2 = list1.ToList<string>();
```
  
  
#### FindIndex
  
```
public void DelUserFromLobbyUserList(UInt64 CharCd)
{
     // List에 LobbyUserInfo의 객체가 들어가 있음
     FindIndex(delegate(LobbyUserInfo user) { return user.CharCd == CharCd; } );
}
```
  
  
#### struct의 멤버 데이터 변경
- class의 경우 특정 인덱스에 있는 class의 멤버 데이터 변경을 임의접근 []로 변경할 수 있다.
  
```
List<class> list = new List<class>();
list.Add(new class(3));
list[0].n = 4;
```
  
- 그러나 struct는 위와 같이 임의 접근을 통해서 값을 변경할 수 없다. 값을 변경하고 싶다면 임시 변수를 만든 후 변경 후 원 데이터를 덮어써야 한다.
  
```
struct S
{
	public int F;

	public S(int f)
	{
		F = f;
	}
}

List<S> list = new List<S>();
list.Add(new S(3));

S tmp = list[0];
tmp.F = 16;

list[0] = tmp;
```
  
  
#### OrderBy
  
```
string[] strs = new string[] { "b", "aaaaa", "cc" };

// 오름 차순으로
var query = strs.OrderBy(s => s);

string[] sortedStrs = query.ToArray();

// 결과 { "aaaaa", "b", "cc" }
```
  
```
string[] strs = new string[] { "b", "aaaaa", "cc" };

// 요소의 Length로 오른차순
var query = strs.OrderBy(s => s.Length);
string[] sortedStrs = query.ToArray();

//결과 { "b", "cc", "aaaaa" }
```
  
```
string[] strs = new string[] { "d", "aaaaa", "cc", "b" };

// 문자열 길이로 오른차순. 동률인 경우는 그대로
var query = strs.OrderBy(s => s.Length).ThenBy(s => s);
string[] sortedStrs = query.ToArray();

//결과 { "b", "d", "cc", "aaaaa" }
```
  
```
string[] strs = new string[] { "b", "aaaaa", "cc" };

// Length 크기로 내림 차순
var query = strs.OrderByDescending(s => s.Length);
string[] sortedStrs = query.ToArray();
//결과 { "aaaaa", "cc", "b" }
```
  
  
### 배열에서 지정한 범위의 요소를 빼내기
- 출처: http://dobon.net/vb/dotnet/programing/arrayslice.html
- 새로운 배열을 만들어서 Array.Copy로 복사
  
```
int[] ary1 = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
int[] ary2 = new int[5];

// 지정한 범위를 복사한다.
Array.Copy(ary1, 2, ary2, 0, 5);

// ary2는 { 2, 3, 4, 5, 6 }로 된다
```
  
- Buffer.BlockCopy 사용 
    - Array.Copy 보다 처리 속도가 빠르다. 단 기본형만 사용할 수 있다.
  
```
int[] ary1 = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
int[] ary2 = new int[5];

// 지정한 범위를 복사한다.
Buffer.BlockCopy(ary1, 2 * 4, ary2, 0, ary2.Length * 4);
```
  
- Skip과 Take 사용
    - LINQ를 사용할 수 있는 곳에서만 사용 가능
  
```
int[] ary1 = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// Skip으로 2개의 요소를 넘은 후 Take로 5개의요소를 얻는다
int[] ary2 = ary1.Skip(2).Take(5).ToArray();


int[] ary1 = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

// SkipWhile에서 2미만의 요소를 skip 하고, TakeWhile로 7미만의 요소를 얻는다.
int[] ary2 = ary1.SkipWhile(i => i < 2).TakeWhile(i => i < 7).ToArray();

// ary2는 { 2, 3, 4, 5, 6 }로 된다
```
  
  
  
### 컬렉션에서 중복이 아닌 값 추출하기
  
```
var a = new[] { 0, 0, 1, 1, 2, 2, 3, 3, 4, 4 };
var list = a.Distinct();
```
  
  
  
### 두 개의 컬력션 합병하기
  
```  
var a = new[] { 0, 0, 1, 1, 2, 2, 3, 3, 4, 4 };
var b = new[] { 0, 2, 4, 6, 8, 10, 12 };
var list = a.Union(b);
```
  
  
  
### 두 개의 컬력션 중 공통부분만 구하기
  
```  
var a = new[] { 0, 0, 1, 1, 2, 2, 3, 3, 4, 4 };
var b = new[] { 0, 2, 4, 6, 8, 10, 12 };
var list = a.Intersect(b);
```
  
  
  
### 컬력션 a에서 b에 포함된 요소를 제거하기
  
```
var a = new[] { 0, 0, 1, 1, 2, 2, 3, 3, 4, 4 };
var b = new[] { 0, 2, 4, 6, 8, 10, 12 };
var list = a.Except(b);
```
  
  
  
### Tuple
  
```
var population = new Tuple<string, int, int, int, int, int, int>(
                           "New York", 7891957, 7781984,
                           7894862, 7071639, 7322564, 8008278);

var primes = Tuple.Create(2, 3, 5, 7, 11, 13, 17, 19);

var number1 = primes.Item1;
var number2 = primes.Item2;
```
  
  
  
### Queue
- 스레드 세이프하게 데이터 접근 하기
  
```
Queue<int> queue = new Queue<int>();
lock (((ICollection)q).SyncRoot)
{
	foreach (int item in q)
	{
		Console.Write("{0} ", item);
	}
}
```
  
  
  
### SortedList
- 용도: 요소 삽입, 삭제 보다도 검색 쪽이 압도적으로 많은 경우 사용한다.
- 계산량: 요소의 삽입, 삭제는 O(n), 검색은 O(log n)
- 내부 구현: 정렬된 배열에 대한 이분검색을 사용한다. 배열은 배열 리스트와 같은 재확보를 한다.
- 기본 정렬은 오름차순
- 기본 사용 방법
  
```
SortedList<int, int> 정렬된_최대값 = new SortedList<int, int>();
int 구간_최대값_구하기ver2(int removeValue, int inputValue)
{
		// key의 값이 1이면 삭제, 1 보다 크면 1 감속
    if (정렬된_최대값[removeValue] == 1)
    {
        정렬된_최대값.Remove(removeValue);
    }
    else
    {
        --정렬된_최대값[removeValue];
    }

		// 이미 있는 key 라면 값 1 증가, 없다면 추가
    if (정렬된_최대값.ContainsKey(inputValue))
    {
        ++정렬된_최대값[inputValue];
    }
    else
    {
        정렬된_최대값.Add(inputValue, 1);
    }

		// 특정 위치의 값을 얻는다. 가장 마지막 위치의 값을 반환
    return 정렬된_최대값.IndexOfKey(정렬된_최대값.Count - 1);
}
```
  
- 커스텀화한 정렬 함수 사용하기
  
```
class UserRankKey
{
    public string UserID;
    public Int64 Score;
}
class UserRankValue
{
    public int Dummy1;
    public int Dummy2;
    public int Dummy3;
}

SortedList<UserRankKey, UserRankValue> RankingList;

class SortedListTestCompare : IComparer<UserRankKey>
{
    public int Compare(UserRankKey x, UserRankKey y)
    {
        if (x.Score < y.Score)
        {
            return -1;
        }
        else if (x.Score > y.Score)
        {
            return 1;
        }
        else
        {
            return 0;
        }
    }
}


if (RankingList.ContainsKey(dummyKeyList[i]))
{
    WriteLog("SortedList: 추가 중 동일 키 발견");
}
else
{
	RankingList.Add(dummyKeyList[i], dummyValueList[i]);
}
```
  
- [What's the difference between SortedList and SortedDictionary?](http://stackoverflow.com/questions/935621/whats-the-difference-between-sortedlist-and-sorteddictionary)
  
  
  
### SortedDictionary<TKey, TValue>
- 용도: 사전에 많은 영역을 차지하고 싶은 않은 경우나 요소 수가 크게 바뀌므로 사전 확보 양을 정하기 어려운 경우에 사용한다.
    - 키 순서대로 요소를 열겨할 수 있다.
- 계산량: 요소의 삽입, 검색, 삭제는 O(log n)
- 내부 구현: 이분검색 트리(레드블랙 트리)
  
  
  
### SortedSet
https://msdn.microsoft.com/ja-jp/library/dd395024(v=vs.110).aspx
  
  
  
### IComparer
http://dobon.net/vb/dotnet/programing/icomparer.html
  
  
  
### 정렬 성능 테스트
- 하드웨어 사양: i7-2600(3.4GHz), 16기가 RAM
- 시간 단위는 초
  
| Collection | 100,000 | 1,000,000 | 5,000,000 | 1,000,000 에서 요소 1개 변경 |
|:-----------|------------:|:------------:|:------------:|:------------:|
| List|  0.03 |  0.48|  3.24|  0.35 ~ 0.04|
| ParallelQuickSort|  0.01|  0.23 ~0.24|  1.4 ~ 1.5|  0.21 ~ 0.27|
| RtRankingLib |  0.51|  7.1|  45|  0.0002 이하|
  
- RtRankingLib: 추가할 때 정렬하므로 한번에 대량의 데이터를 넣을 때는 느림. 그러나 갱신은 엄청나게 빠름
    - https://github.com/jacking75/smrr-server
  
  
  
### ParallelQuickSort
  
```
ParallelQuickSort<ParallelQuickSortTestUserRank>.Sort(PQSTestRankingList.ToArray(), new PQSTestCompare());


class ParallelQuickSortTestUserRank
{
	public string UserID;
	public Int64 Score;
	public int Dummy1;
	public int Dummy2;
}

List<ParallelQuickSortTestUserRank> PQSTestRankingList;

class PQSTestCompare : IComparer<ParallelQuickSortTestUserRank>
{
	public int Compare(ParallelQuickSortTestUserRank x, ParallelQuickSortTestUserRank y)
	{
		if (x.Score < y.Score)
		{
			return -1;
		}
		else if (x.Score > y.Score)
		{
			return 1;
		}
		else
		{
			return 0;
		}
	}
}

//http://elemarjr.net/2012/01/04/parallel-quicksort-em-c/
//https://gist.github.com/ElemarJR/1562593
public static class ParallelQuickSort<T>
{
	public static void Sort(
		T[] input,
		IComparer<T> comparer,
		int maxDepth = 16,
		int minBlockSize = 10000
		)
	{
		InternalSort(input, 0, input.Length - 1,
			comparer,
			0,
			maxDepth,
			minBlockSize);
	}

	private static void InternalSort(
		T[] input,
		int start, int end,
		IComparer<T> comparer,
		int depth,
		int maxDepth,
		int minBlockSize
		)
	{
		if (start >= end) return;
		if (depth > maxDepth || (end - start) < minBlockSize)
		{
			Array.Sort(input, start, end - start + 1, comparer);
		}
		else
		{
			int pivot = PartitionBlock(input, start, end, comparer);

			var left = Task.Factory.StartNew(() =>
				{
					InternalSort(input, start, pivot - 1,
						comparer, depth + 1, maxDepth, minBlockSize);
				});

			var right = Task.Factory.StartNew(() =>
				{
					InternalSort(input, pivot + 1, end,
						comparer, depth + 1, maxDepth, minBlockSize);

				});

			Task.WaitAll(left, right);
		}
	}

	private static void InternalSort2(
		T[] input,
		int start, int end,
		IComparer<T> comparer,
		int depth,
		int maxDepth,
		int minBlockSize
		)
	{
		if (start >= end) return;
		if (depth > maxDepth || (end - start) < minBlockSize)
		{
			Array.Sort(input, start, end - start + 1, comparer);
		}
		else
		{
			int pivot = PartitionBlock(input, start, end, comparer);

			InternalSort2(input, start, pivot - 1,
				comparer, depth + 1, maxDepth, minBlockSize);

			InternalSort2(input, pivot + 1, end,
				comparer, depth + 1, maxDepth, minBlockSize);
		}
	}

	private static int PartitionBlock(
		T[] input,
		int start,
		int end,
		IComparer<T> comparer)
	{
		T pivot = input[start];
		Swap(input, start, end);

		int result = start;
		for (int i = start; i < end; i++)
		{
			if (comparer.Compare(input[i], pivot) <= 0)
			{
				Swap(input, i, result);
				result++;
			}
		}

		Swap(input, result, end);
		return result;
	}

	private static void Swap(T[] input, int first, int second)
	{
		T h = input[first];
		input[first] = input[second];
		input[second] = h;
	}
}
```
  
  
  
### 참고
- [Collection, Array 사용 Tip] (http://emkcsharp.hatenablog.com/entry/2013/Advent)
- [C#/.NET Fundamentals: Choosing the Right Collection Class](http://geekswithblogs.net/BlackRabbitCoder/archive/2011/06/16/c.net-fundamentals-choosing-the-right-collection-class.aspx)
  