---
layout: post
title: 참, 거짓을 반환 하는 함수의 네이밍
published: true
categories: [ETC]
tags: coding
---
참 거짓 값(Boolean, bool)을 반환하는 함수는 is로 시작하는 것이 일반적이라고 생각한다. 적어도 C++ 에서는...  
그러나 영어적으로 is 시작이 어려운 경우도 있다.  
is로 시작할 수 없는 함수 이름의 명명 방식을 생각해 봤다.
  
  
## 존재 하는가?
가장 하기 쉬운 실패가 "존재할까?"를 is에서 시작 하는 패턴이다.  
"존재 하는가?"라는 함수는 아무래도 is로 시작되지 않는다고 생각한다.  
  
isExist 라는 표현을 보는 일이 많지만 개인적으로는 쓰지 않는다.  
exist는 동사이므로 is와 조합하는 것은 영어적으로 있을 수 없다.  
그러나 영어적 올바름보다는 참거짓 값이 is에서 시작하는 것이 중요하다고 생각하는 경우는 잘못된 이름은 아니라고 생각한다.  
충분히 의미도 전해진다.  
  
isExistence 라면 "존재할까?" 보다는 "존재?" 라는 느낌으로 뜻이 모호해진다.  
  
isExisted도 안 된다.  
exist는 자동사이므로 영어적으로 있을 수 없다.  
과거형으로 되어 버리므로 알기 쉬움이라는 의미에서는 isExist 쪽이 더 좋다.  
  
isExisting 나 isExistent는 미묘하지만 역시 이상하다고 생각 된다.  
  
  
기존의 API는 이 표현을 어떻게 쓰고 있을까  
PHP 에서는  
```
bool file_exists ( string $filename )  
```
  
Apple의 NSFileManager 클래스에서는  
```
(BOOL)fileExistsAtPath:(NSString *)path  
```  
  
Microsoft의 File 클래스에서는    
```
bool Exists( string path )  
```
  
Android의 File 클래스에서는  
```
boolean exists()  
```
    
모두 exists를 사용하고 있다.  
소프트웨어의 세계에서는 Apple과 Microsoft과 Google가 검은 색이라과 하면 검은색이다.  
잠자코 따르겠다.  
이처럼 함수 이름의 표현이 어렵다면 세상에 있는 API를 참고하면 좋다.  
비 원어민인 우리들로는 생각하지 않는 정확한 표현이 발견되기도 한다.  
  
  
  
## 함수 명명 방식
참 거짓을 반환하는 함수는 if 문에서 쓰이는 일이 많으므로, 머리에 if를 두고 가장 잘 맞는 표현이 좋다.  
개인적으로는 참 거짓을 반환하는 함수 이름을 생각할 때는 아래의 포맷에 맞추도록 하고 있다.  
  
### if 객체 이름 함수 이름
"항목이 선택 중이라면" 라면 "if item is selected"이므로 함수 이름은 item.isSelected() 이다.  
마찬가지로 "항목이 존재하면"이라면 "if item exists"이므로 item.exists() 이다.  
"명단에 항목이 포함됐다면"이라면 "if list contains item"이므로 list.contains(item) 또는 list.containsItem(item) 으로 되고,  
"목록 항목을 삭제할 수 있다면"이라면 "if list can remove item"이므로 list.canRemove(item) 또는 list.canRemoveItem(item) 이다.  
마지막 예는 좀 수상한데 이런 느낌으로 함수 이름을 생각하고 있다.  
이렇게 하면 함수 이름이 exist 가 아닌 exists로 되어 있는 점도 납득할 수 있다.  
  
isEnabled()는 일반적인 함수 이름이지만 "if is enabled"는 영어적으로 이상하다.  
이 경우는 isEnabled()가 this->isEnabled()인 것을 생각한다.  
"if this is enabled"면 위화감은 없다.  
또 Item 클래스에서 isItemEnabled() 이라는 함수 이름으로 하려면 "if this(Item)is item enabled"라는 위화감 있는 영어로 되어 버려서  함수명이 부적절하다는 것을 알 수 있다고 생각한다.  
  
물론 이 포맷이 맞지 않는 경우도 많이 있다.  
  
예를 들어 Item 클래스의 멤버 변수에 name 이라고 하는 문자열이 있고, 이 문자열이 빈 여부를 판단하는 경우는 isNameEmpty()라는 이름을 붙인다.  
포맷에 적용하면 "if this is name empty"로 이상하다.  
this를 생략해도 "if is name empty"로 이것도 이상하다.  
그래도 함수 이름은 isNameEmpty()이 일반적이라고 생각한다.  
  
isNameEmpty()가 아닌 hasEmptyName() 쪽이 영어적으로 좋다는 조언을 받았다.  
"if this has empty name"가 되므로 포맷에도 해당된다.  
  
  
### is로 시작 하지 않는 표현
is 이외에서 시작하는 경우는 동사나 조동사를 사용한다.  
이 이외의 품사로 시작하는 것은 이상하다.  
  
동사 시작에서 참 거짓을 반환하는 표현의 예:  
- exists(존재할까)
- contains(포함되어 있는가)
- has( 가지고 있는가)
- needs(필요한가)
  
조동사 시작에서 참 거짓을 반환하는 표현의 예:  
- can(가능한가)
- should( 해야 하는가)
- need( 할 필요가 있을까)
  
  
### can=is, able
가능한지 여부는 can 시작으로 한다.  
   
- canRemoveItem(item)
- canUpdate()
- canStartTimer(timer)
  
그러나 영어적으로는 can은 is, able로 바꾸는 것도 있다(아마두).  
- isItemRemovable(item)
- isUpdatable()
- isTimerStartable(timer)
  
  
### is 이외의 be 동사를 사용하고 싶은 경우
"모든 아이템이 삭제가 됐는지"를 확인하는 함수를 작성한 경우 어떤 이름으로 해야 할까?  
생각 없이 갑자기 생각 나는 것은 아래의 2개이다.  
  
- isAllItemsRemoved
    - is로 시작됐고 참거짓을 반환하는 함수 이름으로 하면 느낌은 좋지만, 명사가 복수형이라서 영어적으로는 틀리다.
- areAllItemsRemoved
    - 영어적으로 옳지만 are 에서 시작하는 것에 위화감이 있다.
  
"함수 이름적으로 기분 좋은 표현을 우선할 것인가" "영어적으로 올바른 표현을 우선할 것인가"는 점에 대해서는 의견이 갈릴 것으로 생각한다.  
나 개인으로는 영어적 올바름을 우선하므로 상기 2개 중 하나를 꼽으라면 are 쪽이다.  
  
단지 실제로는 "모든 아이템이 삭제가 됐는지"을 함수로 할 경우 상기 2가지가 아니다 아래의 함수 이름으로 하려고 한다.  
- isEveryItemRemoved
    - 세세한 것을 말하면, all과 every는 영어적 뉘앙스가 다르다고 생각하지만, is 시작이고, 영어적으로도 올바른 것으로 알기 쉽다.
    - 다만 Every가 "모든"이라는 뜻이라는 것은 All 정도로 인지되지 않은 것 같기도 하다.
- itemExists
    - "존재할까?"에 표현을 바꾼 것으로써 기분 좋은 함수명으로 한다.
  
이런 느낌으로 함수 이름을 헷갈리면 다른 표현을 검토하는 것도 좋은 방법이라고 생각한다.  
  
is vs are의 논의는 아래에서도 하고 있다.  
https://stackoverflow.com/questions/12960554/java-boolean-getters-is-vs-are  
https://stackoverflow.com/questions/2691463/is-or-are-to-prefix-boolean-values  
  
  
## 참 거짓을 설정하는 함수
지금까지는 참 거짓을 반환하는 함수(이하, getter)를 썼는데, 마지막으로 참 거짓을 설정하는 함수(이하, setter)에 대해서도 남긴다.  
   
기본적으로 is를 set으로 바꾼다.  
getter가 isSelected 라면 setter는 setSelected 으로 한다.  
꼭 isXXX 와 setXXX가 짝을 짓도록 한다.  
이렇게 하는 것이 알기 쉽다.  
  
is로 시작되지 않으면 머리에 set을 붙인다.  
canRemove 라면 setCanRemove 으로 한다.  
exists 라면 setExists 으로 한다.  
영어적으로는 이상한 표현인데, 영어의 정확성보다 함수의 관계성의 알기 쉬움을 우선한다.  
  
이 근처의 룰은 Apple의 Naming Guidelines을 참고로 한다.  
https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html#//apple_ref/doc/uid/20001282-1004202  
  
내가 함수나 변수의 이름을 진지하게 생각하게 된 것이 iOS앱 개발에 종사하고 나서니까, 나의 네이밍 사상은 Apple의 영향이 강하다.  
  
그래서 Apple의 Naming Guidelines를 추천한다!
(Objective-C 이므로 주의)  
https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html  
  
<br><br>  
  
출처: https://qiita.com/yskszk/items/5a7f99c974773f03a82a
  