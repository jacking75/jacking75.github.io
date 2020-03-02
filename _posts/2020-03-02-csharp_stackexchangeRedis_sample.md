---
layout: post
title: StackExchange.Redis - 간단 사용
published: true
categories: [.NET]
tags: c# .net StackExchange.Redis redis
---
StackExchange.Redis 사용 방법 정리.  
  
## 문자열 저장 및 읽기
[출처](http://genpaku3.hatenadiary.jp/entry/2017/11/27/002229 )  
```
using System;
using StackExchange.Redis;
using System.Threading;

namespace RedisSample
{
    class Program
    {
        static void Main(string[] args)
        {
            ConnectionMultiplexer redis = ConnectionMultiplexer.Connect("localhost");

            // 데이터베이스 얻기
            IDatabase cache = redis.GetDatabase();

            // 값 저장
            cache.StringSet("key:jp:hello", "안녕하세요");
            cache.StringSet("key:jp:goodbye", "안녕");

            // 저장한 값을 얻은 후 출력
            Console.WriteLine(cache.StringGet("key:jp:hello"));
            Console.ReadLine();
            Console.WriteLine(cache.StringGet("key:jp:goodbye"));
            Thread.Sleep(1000);
        }
    }
}
```
  
  
## 비동기로 문자열 저장 및 읽기 
[출처](http://genpaku3.hatenadiary.jp/entry/2017/11/27/002229 )   
```
using System;
using StackExchange.Redis;
using System.Linq;
using System.Diagnostics;

namespace RedisSample
{
    class Program
    {
        static void Main(string[] args)
        {

            // 관리자 명령어를 사용하기 위해 접속 문자열에 allowAdmin=true"를 추가
            ConnectionMultiplexer redis = ConnectionMultiplexer.Connect("localhost,allowAdmin=true");

            IDatabase cache = redis.GetDatabase();

            // 데이터 베이스 초기호
            var server = redis.GetServer("localhost", 6379);
            server.FlushAllDatabases();

            // key 생성을 위해 숫자를 준비
            var range = Enumerable.Range(1, 10000);

            // 처리 시간을 측정
            Stopwatch sw = new Stopwatch();
            sw.Start();

            foreach (var index in range)
            {
                // 비동기로 오브젝트를 추가
                cache.StringSetAsync($"number:{index}", $"result:{index}");
            }
            //모든 비동기 처리를 기다린다
            cache.WaitAll();
            sw.Stop();

            //사용 중의 메모리를 얻는다
            var usedMemory = server.Info().Where(group => group.Key.Equals("Memory")).First().Where(c => c.Key.Equals("used_memory")).First();
            Console.WriteLine($"비동기 처리 처리 결과 후/ 사용 메모리 {usedMemory.Key}:{usedMemory.Value} / 처리 시간 {sw.ElapsedMilliseconds} ms / Key의 수 {server.Keys().Count()}");

            Console.ReadLine();
        }
    }
}
```
  