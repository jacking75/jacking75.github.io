---
layout: post
title: C# - Linq
published: true
categories: [.NET]
tags: c# .net Linq
---
## 기본
- Linq는 아래의 2개의 데이터 소스를 대상으로 한다.
    - IEnumerable<T>
    주로 온 메모리 데이터 소스를 나타내는 Interface
    LINQ to Object, LINQ to XML
    - IQueryable<T>
    주로 외부 데이터 소스를 나타내는 interface
    LINQ to SQL, LINQ to Entity
  
  
  
## Where
- 조건을 만족하는 요소를 추출
  
```
var result = new List<Character>();

foreach (var character in characters)
{
    if (character.Job == "Soldier")
    {
        result.Add(character);
    }
}
```  
=>  
```
var result = characters.Where(c => c.Job == "Soldier");
```
  
- 요소의 특정 필드만 추출
    
```
var result = new List<string>();

foreach (var character in characters)
{
    result.Add(character.Name);
}
```  
=>  
```
var result = characters.Select(c => c.Name);
```
  
- Where 메소드를 체인으로 AND 검색
    - Where 메소드를 체인하면 코드는 실행 시에는 자동적으로 하나로 묶여진다(Where 메소드에 한정되지 않고 IEnumerable<T> 인터페이스를 반환하는 확장 메소드도 같음)
  
```
// Where 메소드를 체인하여 ADN 검색
IEnumerable<string> = sampleData.Where(item => item.ContainsEx("ぶた"))
                          .Where(item => item.ContainsEx("まつり"));
WriteItems("LIKE '%ぶた%' AND LIKE '%まつり%'", AND検索1);
```
  
  
  
## (에외없이)첫번째 요소만 찾기  FirstOrDefault
- 대상이 되는 리스트에서 요소를 찾지 못한 경우 default 값을 반환한다. 요소가 참조형인 경우 default 값은 null이 된다.
  
```  
Uri[] urls = new Uri[]
{
  new Uri("http://www.testserver.com"),
  new Uri("http://www.testserver.com:8080"),
  new Uri("http://www.dbserver.com"),
  new Uri("http://www.dbserver.com:8080"),
  new Uri("http://www.proxyserver.com"),
};

// 「devserver」를 포함하는 Uri을 하나 취득
Uri oneDevServer = (from n in urls
                    where n.Host.Contains("devserver")
                    select n).FirstOrDefault();
```
  
  
  
## (예외 없이)조건을 만족하는 마지막 요소를 취득 LastOrDefault

```
Character result = null;

for (int i = characters.Length - 1; i >= 0; i--)
{
    if (characters[i].Job == "Soldier")
    {
        result = characters[i];
        break;
    }
}
```
  
=>  
  
```  
var result = characters.LastOrDefault(c => c.Job == "Soldier");
```
  
  
  
## 집계 Sum
- 기본
  
```
var nums = Enumerable.Range(1, 10);
var sum = nums.Sum();
Console.WriteLine(sum); // 55
```
  
- 특정 데이터만 집계
  
```
var nums = new int[] { 1, -1, 2, -2, 3, -3, 4, -4, 5, -5, };
var sum = nums.Where(n => n > 0).Sum(); // 0 보다 큰 수만 집계한다.
Console.WriteLine(sum); // 15
```
  
```
var nums = new int[] { 1, -1, 2, -2, 3, -3, 4, -4, 5, -5, };
var sum = nums.Sum(n => n > 0 ? n : 0);
Console.WriteLine(sum); // 15
```
  
- null 값은 자동으로 제외
  
```
var nullableNums = new int?[] { 1, null, 2, null, 3, null, 4, null, 5, null, };
var sum = nullableNums.Sum();
Console.WriteLine(sum); // 15
```
  
- 오브젝트의 특정 필드의 값만 집계
  
```
var data = new System.Collections.Generic.List<Data>()
                {
                  new Data(){ Text="data1", Number=1 },
                  new Data(){ Text="data2", Number=2 },
                  new Data(){ Text="data3", Number=3 },
                  new Data(){ Text="data4", Number=4 },
                  new Data(){ Text="data5", Number=5 },
                };

var sum1 = data.Select(d => d.Number).Sum();
Console.WriteLine(sum1); // 15

var sum2 = data.Sum(d => d.Number);
Console.WriteLine(sum)2; // 15
```
  
  
  
##평균값을 구한다  Average
  
```
var result = 0;

foreach (var item in items)
{
    result += item.Price;
}
result /= items.Count;
```
  
=>  
```
var result = items.Average(c => c.Price);
```
  
  
  
## 정렬 OrderBy
- 반환 값은 IEnumerable
- 오름순. 내림순은 OrderByDescending
  
```
public class Word {
  public string data;
  public Word(string s) { data = s; }
}

Word[] wordArray = {
  new Word("むつき"),
  new Word("きさらぎ"),
  new Word("やよい"),
  new Word("ふみづき"),
  new Word("はづき"),
  new Word("かんなづき"),
  new Word("しもつき"),
};

// data 필드의 문자 길이로 정렬
wordArray = wordArray.OrderBy(n => n.data.Length)
                     .ToArray();

// data 필드의 문자열의 길이 + 아이우에오 순으로 정렬
wordArray = wordArray.OrderBy(n => n.data.Length)
                     .ThenBy(n => n.data)
                     .ToArray();
```
  
  
  
## 최소값, 최대값
- null은 자동으로 제외
  
```
var nums = Enumerable.Range(1, 10);
var min = nums.Min();
var max = nums.Max();
Console.WriteLine("{0},{1}", min, max); // 1, 10
```
  
- 최대 값을 가진 요소를 취득
  
```
// 첫번째
int max = characters.Max(c => c.Power);
var result = characters.First(c => c.Power == max);

// 모든 것
int max = characters.Max(c => c.Power);
var result = characters.Where(c => c.Power == max);
```
  
  
  
## 개수 Count
- 특정 조건의 요소만 카운팅
  
```
string s = "この文字列の中の「の」の数は？";
Console.WriteLine(s.Count(c => c == 'の'));
```
  
  
  
## SKip, Take
- Skip으로 2개의 요소를 넘은 후 Take로 5개의요소를 얻는다
  
```
int[] ary1 = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
int[] ary2 = ary1.Skip(2).Take(5).ToArray();
```
  
- SkipWhile에서 2미만의 요소를 skip 하고, TakeWhile로 7미만의 요소를 얻는다.
  
```
int[] ary1 = new int[] { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
int[] ary2 = ary1.SkipWhile(i => i < 2).TakeWhile(i => i < 7).ToArray();
// ary2는 { 2, 3, 4, 5, 6 }로 된다
```
  
  
  
## 모든 요소가 조건을 만족하는가 All
  
```
var result = true;

foreach (var character in characters)
{
    if (character.Job != "Soldier")
    {
        result = false;
        break;
    }
}
```
  
=>
  
```
var result = characters.All(c => c.Job == "Soldier");
```
  
  
  
## 일부 요소가 조건을 만족하는가 Any
  
```
var result = false;

foreach (var character in characters)
{
    if (character.Job == "Soldier")
    {
        result = true;
        break;
    }
}
```
  
=>
  
```
var result = characters.Any(c => c.Job == "Soldier");
```
  
  
  
## 지정된 형에 일치하는 요소를 추출
- OfType 메소드를 사용한다.
  
```
foreach (var character in characters)
{
    var result = character as Alchemist;
    if (result != null)
    {
        result.Alchemy();
    }
}
```
  
=>
  
```
foreach (var result in characters.OfType<Alchemist>())
{
    result.Alchemy();
}
```
  
  
  
## 배열을 특정 값으로 초기화 한다
  
```
bool[] flags = new bool[100];
for (int i = 0; i < flags.Length; i++)
{
    flags[i] = true;
}
```
  
=>
  
```
bool[] flags = Enumerable.Repeat(true, 100).ToArray();
```
  
  
  
## 연순
  
```
var result = items.Reverse();
```
  
  
  
## 중복을 제거한다.
  
```
var result = names.Distinct();
```
  
