---
layout: post
title: C++ - Google Test/Mock 기능 정리
published: true
categories: [C++]
tags: c++ GTest GMock
---
[원문](https://qiita.com/hakua-doublemoon/items/0b81cc069b53431df20f )  
  
## 매크로
  
### TEST(), TEST_F(),TEST_P()
https://github.com/google/googletest/blob/master/googletest/docs/primer.md#simple-tests  

```
TEST(TestSuiteName, TestName) {
  ... test body ...
}
```
  
- 테스트 케이스의 대 항목dls 이름과 소항목인 이름을 입력한다.
    - 여러 테스트가 있는 경우는 대 항목 이름과 소항목 이름의 조합이 중복되지 않도록 하지 않으면 빌드 오류가 발생한다.
- `TEST_F`에서 첫 번째 인수는 테스트 픽처로 정의된 클래스 이름으로 한다.
- `TEST_P`는 고급 기술로 하나의 테스트 매개 변수를 바꿔가면서 여러 번 수행 할 수 있다.
    - http://opencv.jp/googletestdocs/advancedguide.html#adv-how-to-write-value-parameterized-tests
  
  
### ASSERT_EQ, EXPECT_EQ
http://opencv.jp/googletestdocs/primer.html#primer-binary-comparison  
  
- 첫 번째 인수가 기대 값, 둘째 인수가 실제 값. 함수를 실행했을 때 반환 체크 등에 사용할 수 있습니다.
- EXPECT_NE나 EXPECT_TRUE같은 친척도 있습니다
- 상황이 예상과 다른 경우 즉시 테스트가 종료되는 것이 ASSERT_*계속하는 것이 EXPECT_*있습니다.
  
  
### MOCK_METHOD*, MOCK_CONST_METHOD*
- mock 메소드의 정의에 사용한다. `*` 부분은 인수의 수이다.
    - 인수의 수는 10을 넘으면 죽는다. 인수가 많은 함수가 많은 프로젝트에서는 대책이 필요하다.
  
  
### EXPECT_CALL
- Mock의 메소드가 호출되는 것을 선언한다. 첫 번째 인수가 Mock화된 클래스의 인스턴스(포인터가 아닌 실체), 두 번째 인수가 메소드 이름
    - 메소드 이름은 인수를 세트로 지정한다. 인수가 일치하지 않는 호출된 경우는 호출 되지 않는다.
    - 아래의 ::testing::_를 사용해서 인수의 체크를 없애준다
- 아래의 NiceMock이 아닌 경우, EXPECT 되지 않은 Mock 메소드 호출은 경고된다.(테스트는 성공하기도 한다). EXPECT한 호출이 아니었다면 테스트는 실패한다.
  
  
### ::testing::_
https://github.com/google/googletest/blob/master/googlemock/docs/cheat_sheet.md#wildcard  
  
- 정확한 정의로는 Matcher의 wildcard이며, 이 이름대로 임의의 인수를 허용한다
- Matcher는 위의 페이지에 여러 있으므로 봐두면 도움이 될듯
  
  
### Times
http://opencv.jp/googlemockdocs/fordummies.html#cardinalities  
  
- EXPECT_CALL 멤버와 비슷한 형태로 사용하고 호출 횟수를 지정할 수 있다. cardinality(기수)라고 한다.
- ::testing::AtLeast(n)(n은 0 이상의 정수)로 횟수 체크를 느슨하게 할 수 있다
- 지정된 횟수까지 호출하지 않으면 테스트가 실패한다
- Times(0)로 함으로써 이 메소드가 호출되지 않음을 선언 할 수 있다
- Times(1)은 생략 할 수 있다.
  
  
### WillOnce, WillRepeatedly
http://opencv.jp/googlemockdocs/fordummies.html#fordummies-general-syntax  
  
- EXPECT_CALL 멤버 형태로 사용하고, 호출 할 때의 action을 정의 할 수 있다
    - action은 주로 반환 값 지정이다. 반환 값이 없는 메서드는 이것은 할 수 없다.
- `Will**`은 `Times`의 뒤에 써야 한다.
- `WillOnce`는 여러 번 사용할 수 있으며 이때마다의 반환 값을 action을 바꿀 수 있다.
  
```
using ::testing::Return;...
EXPECT_CALL(turtle, GetX())
    .Times(5)
    .WillOnce(Return(100))
    .WillOnce(Return(150))
    .WillRepeatedly(Return(200));
```
  
  
### ::testing::Return
https://github.com/google/googletest/blob/master/googlemock/docs/cheat_sheet.md#actions-actionlist  
  
- 위의 `Will**`의 action으로 값을 반환하는 동작을한다
- action도 위의 페이지처럼 여러 종류가 있다. 
  
  
### RetiresOnSaturation
http://opencv.jp/googlemockdocs/fordummies.html#expectation-sticky  
  
- EXPECT_CALL 멤버 형태로 사용하고, 이 EXPECT_CALL의 역할이 끝나면 더 이상 EXPECT_CALL를 보이지 않도록 지정할 수 있다.  
>>> Google Mock의 Expectation가 default로는 "sticky"임을 나타낸다
  
  
### ::testing::Invoke
http://opencv.jp/googlemockdocs/cheatsheet.html#cheatsheet-using-afunction-or-functor  
  
- 아래와 같이 함수를 action으로 정의 할 수 있다. mock 대신 fake적인 사용법이다.  
``` 
double Distance(Unused, double x, double y) { return sqrt(x*x + y*y); }
...
EXPECT_CALL(mock, Foo("Hi", _, _)).WillOnce(Invoke(Distance));
```
  
- action에 Invoke로 등록하는 함수는 Mock의 메소드와 같은 함수의 형태(또는 void *와 같은 호환성 이있는 어떤 타입이어야한다.
    - 사용하지 않는 인수는 Unused로 할 수 있다. 위의 예에서도 사용하고 있다.
- InvokeWithoutArgs도 있는 것 같다. 이것은 인수를받지 않는 대신 일체의 인수를 무시할 수 있다.
  
  
### NiceMock<>
http://opencv.jp/googlemockdocs/cookbook.html#nice-strict  
  
- 이것으로 선언된 Mock은 이 Mock 메소드가 EXPECT_CALL로 설정되기 전에 호출되어도 Assert 등을 발행하지 않는다.
    - "우선 EXPECT_CALL을 명시하고 mock의 동작을 지정하자"
    - "매우 신중하게 이용해야 한다"  "버그가 경고되지 않고 간과 될 수 있다"
    - 아무래도 좋다 클래스에서만 사용할 수 있도록 하는 것을 저도 추천한다
- 친척인 StrictMock도 있다.
  
  
  
## 문법적인 것
  
### 테스트 픽쳐
https://github.com/google/googletest/blob/master/googletest/docs/primer.md#test-fixtures-using-the-same-data-configuration-for-multiple-tests  
  
- ::testing::Test에서 파생된 클래스를 정의하고 픽쳐를 만들 수 있다.
- 구체적으로는 픽처에서 만든 인스턴스를 테스트 케이스 간에 공유하거나 할 수 있다.
  
  
### RUN_ALL_TESTS와 main 함수
http://opencv.jp/googletestdocs/primer.html#primer-invoking-the-tests  
http://opencv.jp/googlemockdocs/fordummies.html#fordummies-using-mocks-in-tests  
  
- 픽처 및 mock를 사용하는 경우에는 초기화로 main()가 필요하다.
- main()을 준비한 경우는 main()의 return으로 RUN_ALL_TESTS()을 실행하게 하면 좋을 것 같다.
    - RUN_ALL_TESTS에서 모든 테스트를 실행한다. 이때 Setup()와 TearDown()가 각 테스트마다 실행되는 것에 주의가 필요한다.
  
  
### ::testing::InitGoogleMock
Mock을 사용하는 경우 초기화를 하도록 한다.  
  
  
### 여러 Exception ( EXPECT_CALL)
http://opencv.jp/googlemockdocs/fordummies.html#fordummies-using-multiple-expectations  
  
- 여러 matcher(match 조건)에 대한 EXPECT_CALL을 미리 써둘 수 있다.  
  
```  
EXPECT_CALL(turtle, Forward(_));  // #1
EXPECT_CALL(turtle, Forward(10))  // #2
    .Times(2);
```
  
- 그러나 여러 Exception은 독특한 동작을 하므로 테스트에 고생 할 수 있다 :
    - Match의 판정은 마지막에 정의된 순서로 하는 것 같다
    - 같은 matcher 대해 여러 EXPECT_CALL을 작성하고 마지막으로 선언한 것으로 덮어 쓰거나한다다
- 같은 Match 조건에서 여러 동작을 시키고 싶은 경우는 WillOnce을 여러번 켜고, ::testing::Invoke을 사용한다
  
  
### InSequence
http://opencv.jp/googlemockdocs/fordummies.html#fordummies-ordered-vs-unordered  
  
- 메소드의 호출 순서를 확인할 수 있다
- 또 다른 예는 EXPECT_CALL에 붙인 순서를 정의하는 방법도 써 있다.
-  순서와 관련하여 After 라고 한다.
  
  
### ON_CALL과 기본 action
http://opencv.jp/googlemockdocs/cheatsheet.html#action  
  
- ON_CALL에서 메소드의 default action을 지정하는 것이다
- Times 미지정되지 않은 WillRepeatedly에서도 같은 것을 할 수 있지만 ON_CALL를 사용하는 것이 본래의 단계이다
    - DefaultValue<T>::Set(value)라는 것도 있는 것 같다.
  