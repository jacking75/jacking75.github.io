---
layout: post
title: 좋은 함수 이름을 짓기 위한 참고 정보
published: true
categories: [ETC]
tags: coding
---
## 참 거짓을 돌려주는 메소드
  
| 장소   | 단어   | 의미                                                  | 예            |
|--------|--------|-------------------------------------------------------|---------------|
| Prefix | is     | (오브제그트가)대기 하는 상태가 되어 있나?             | isChecked     |
| Prefix | can    | (오브제그트가)기대하는 동작을 할 수 있나?             | canRemove     |
| Prefix | should | (호출 측이)어떤 명령을 실행한쪽이 좋나                | shouldMigrate |
| Prefix | has    | (오브제그트가)기대하는 데이터 프로퍼티를 가지고 있나? | hasObservers  |
| Prefix | needs  | (호출 측이)어떤 명령을 실행할 필요가 있는가?          | needsMigrate  |
  
<br>  
  
## 필요에 의해서만 실행할 처리를 하는 메소드  
  
| 장소   | 단어      | 의미                                                              | 예                     |
|--------|-----------|-------------------------------------------------------------------|------------------------|
| Suffix | IfNeeded  | 필요하면 실행하고, 필요하지 않으면 아무것도 안한다                | drawIfNeeded           |
| Prefix | might     | 위와 같음                                                         | mightCreate            |
| Prefix | try       | 실행을 해보고, 실패한 경우는 예외를 던지거나 에러 코드를 반환한다 | tryCreate              |
| Suffix | OrDefault | 실행해 보고, 실패한 경우는 기본 값을 반환한다                     | getOrDefault           |
| Suffix | OrElse    | 실행해 보고, 실패한 경우는 인수로 지정한 값을 반환한다            | getOrElse              |
| Prefix | force     | 강제적으로 실행해 본다. 에러는 예외 없이 반환 값으로 나타낸다.    | forceCreate, forceStop |
  
<br>  
  
## 비동기 처리와 관련된 메소드
  
| 장소           | 단어         | 의미                                           | 예                    |
|----------------|--------------|------------------------------------------------|-----------------------|
| Prefix         | blocking     | 스레드를 블럭하는 메소드                       | blockingGetUser       |
| Suffix         | InBackground | 백그라운드 스레드로 실행되는 메소드            | doInBackground        |
| Suffix         | Async        | 비동기 메소드                                  | sendAsync             |
| Suffix         | Sync         | (대응하는 비동기 메소드가 존재하는)동기 메소드 | sendSync              |
| Prefix or Stem | schedule     | job 이나 태스크를 큐에 쌓는다                  | schedule, scheduleJob |
| Prefix or Stem | post         | 위와 같음                                      | postJob               |
| Prefix or Stem | execute      | 비동기 처리를 실행한다                         | execute, executeTask  |
| Prefix or Stem | start        | 비동기 처리를 실행한다                         | start, startJob       |
| Prefix or Stem | cancel       | 비동기 처리 실행을 막는다                      | cancel, cancelJob     |
| Prefix or Stem | stop         | 비동기 처리 실행을 막는다                      | stop, stopJob         |
  
<br>  
  
## 콜백 메소드
  
| 장소   | 단어   | 의미                                        | 예           |
|--------|--------|---------------------------------------------|--------------|
| Prefix | on     | 무엇인가 일어났을 때에 실행된다             | onCompleted  |
| Prefix | before | 무엇인가가 일어나기 전에 실행된다           | beforeUpdate |
| Prefix | pre    | 무엇인가가 일어나기 전에 실행된다           | preUpdate    |
| Prefix | will   | 무엇인가가 일어나기 전에 실행된다           | willUpdate   |
| Prefix | after  | 무엇인가 일어난 후에 실행된다               | afterUpdate  |
| Prefix | post   | 무엇인가 일어난 후에 실행된다               | postUpdate   |
| Prefix | did    | 무엇인가 일어난 후에 실행된다               | didUpdate    |
| Prefix | should | 무엇인가 일어나도 좋은지 확인할 때 실행된다 | shouldUpdate |
  
<br>  
  
## 컬렉션 조작에 관한 메소드
  
| 단어     | 의미                                            | 예         |
|----------|-------------------------------------------------|------------|
| contains | 지정한 것과 같은 오브젝트를 가지고 있는가?      | contains   |
| add      | 추가한다                                        | addJob     |
| append   | 추가한다                                        | appendJob  |
| insert   | n번째에 추가한다                                | insertJob  |
| put      | key 에 대응하는 요소를 추가한다                 | putJob     |
| remove   | 요소를 삭제한다                                 | removeJob  |
| enqueue  | 행열 끝에 추가한다                              | enqueueJob |
| dequeue  | 행열 선두를 빼서 제거한다                       | dequeueJob |
| push     | 스택 선두에 추가한다                            | pushJob    |
| pop      | 스택 선두를 빼서 제거한다                       | popJob     |
| peek     | 스택 선두를 빼낸다(스택에서 삭제는 하지 않는다) | peekJob    |
| find     | 조건에 맞는 것을 찾는다                         | findById   |  
  
<br>  
  
## 상태에 관한 메소드
  
| 단어     | 의미                                                                               | 예             |
|----------|------------------------------------------------------------------------------------|----------------|
| ensure   | 기대하는 상태인지 조사하고, 그렇지 않은 경우는 예외를 던지거나 에러코드를 반환한다 | ensureCapacity |
| validate | 올바른 상태인지 조사하고, 그렇지 않은 경우는 예외를 던지거나 에러코드를 반환한다   | validateInputs |
  
<br>  
  
## 오브젝트 라이프사이클을 다루는 메소드
  
| 단어       | 의미                            | 예         |
|------------|---------------------------------|------------|
| initialize | 초기화. 지연 초기화 메소드로드. | initialize |
| abandon    | 소멸자의 대체                   | abandon    |
| destroy    | 소멸자의 대체                   | destroy    |
| dispose    | 소멸자의 대체                   | dispose    |
  
<br>  
  
## 데이터에 관한 메소드
  
| 단어   | 의미                                                            | 예            |
|--------|-----------------------------------------------------------------|---------------|
| create | 새롭게 만든다                                                   | createAccount |
| new    | 새롭게 만든다                                                   | newAccount    |
| from   | 기존의 것에서 새롭게 만든다. 혹은 다른 데이터에서 새롭게 만든다 | fromConfig    |
| to     | 변환한다                                                        | toString      |
| update | 기존의 것을 덮어쓴다                                            | updateAccount |
| load   | 읽기                                                            | loadAccount   |
| fetch  | (리모트에서)읽기読み込む                                        | fetchAccount  |
| delete | 삭제한다                                                        | deleteAccount |
| remove | 삭제한다                                                        | removeAccount |
| save   | 보존한다                                                        | saveAccount   |
| store  | 보존한다                                                        | storeAccount  |
| commit | 보존한다                                                        | commitChange  |
| apply  | 보존・적용한다                                                  | applyChange   |
| clear  | 데이터를 지운다. 혹은 초기화 상태로 되돌린다                    | clearAll      |
| reset  | 데이터를 지운다. 혹은 초기화 상태로 되돌린다                    | resetAll      |
  
  
<br>  
  
출처: https://qiita.com/KeithYokoma/items/2193cf79ba76563e3db6  

