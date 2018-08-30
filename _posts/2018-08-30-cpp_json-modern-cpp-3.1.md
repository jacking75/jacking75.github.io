---
layout: post
title: C++ - JSON for Modern C++ 이 버전 3.1로
published: true
categories: [C++]
tags: c++ json json-modern-cpp
---
[원문](https://www.infoq.com/news/2018/02/json-modern-cpp-3.1)    
  
JSON for Modern C ++ 3.1 에서는 Universal Binary JSON(UBJSON) 기능 지원 및 JSON Merge Patch가 새롭게 추가 되었다.  
  
UBJSON는 JSON for Modern C++에서 지원하는 여러 가지 형식의 하나로, 인코딩 크기 감소 및 디코딩 속도를 목적으로 한다.  
UBJSON 외에는 CBOR 이나 MessagePack을 지원하고 있다.  
각각 독자의 장점을 가지고 있기 때문에 어느 것이 올바른 선택인지는 표현하는 데이터의 종류에 따라 달라진다.  
3개 중 UBJSON만 완전한 바이너리 포맷이다. 즉, 모든 JSON 값을 UBJSON으로, 모든 UBJSON 값을 JSON으로 변환이 가능하다.  
  
JSON Merge Patch는 2개의 JSON 문서 사이의 차이를 선언적으로 기술하는 것을 목적으로 한 포맷이다.  
HTTP PATCH 동사와 함께 사용하기 위한 것이지만, HTTP PUT가이 리소스 전체를 대체 하는 것에 대해 부분적인 업데이트를 가능하게 한다.  
JSON Merge Patch를 사용하면 JSON의 일부를 정의하고 이를 서버 측에서 병합 할 수있다.  
원 JSON 문서에 대한 일련의 패치 작업 사양에 의존하는 기본적인 JSON Patch 포맷보다 이쪽이 편리하다. 예를 들어,  
```
// a JSON value
json j_document = R"({
  "a": "b",
  "c": {
    "d": "e",
    "f": "g"
  }
})"_json;

// a JSON patch (RFC 6902)
json j_patch_1 = R"([
  { "op": "replace", "path": "/a", "value": "z" },
  { "op": "remove", "path": "/f"}
])"_json;


// a JSON Merge patch (RFC 7386)
json j_patch_2 = R"({
  "a":"z",
  "c": {
    "f": null
  }
})"_json;
```  
JSON for Modern C ++ 라이브러리는 JSON 데이터를 마치 일반적인 데이터 형식처럼 취급 할 수있는 직관적인 구문을 제공하기 위한 것이다.  
예를 들어 다음과 같이 기술하는 것으로 개체를 인스턴스화 할 수있다.  
```
json j2 = {
  {"pi", 3.141},
  {"happy", true},
  {"name", "Niels"},
  {"nothing", nullptr},
  {"answer", {
    {"everything", 42}
  }},
  {"list", {1, 0, 2}},
  {"object", {
    {"currency", "USD"},
    {"value", 42.99}
  }}
};
```  
마찬가지로 문자열 리터럴 __json을 덧붙이면 JSON 문자열의 디코딩이 가능하다.  
```
auto j2 = R"(
  {
    "happy": true,
    "pi": 3.141
  }
)"_json;
```  
  
다른 장점은 전체가 하나의 헤더 파일 json.hpp에 담겨 있으며, 외부 라이브러리 및 종속성을 전혀 필요로 하지 않는 데 따른 높은 통합성을 제공한다.  
또한 유닛 테스트를 통해 100%의 코드 커버리지를 실현함과 동시에 메모리 누수도 없다라는 것읻다.  
    
