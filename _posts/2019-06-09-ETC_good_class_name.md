---
layout: post
title: 좋은 클래스 이름을 짓기 위한 참고 정보
published: true
categories: [ETC]
tags: coding
---
## 데이터 소스를 취급하는 레이어
  
| 이름                               | 보충                                                             | 예                                                |
|------------------------------------|------------------------------------------------------------------|---------------------------------------------------|
| Client                             | HttpClient 등 Server-Client 의미로 사용하는                      | TwitterApiClient, QiitaApiClient                  |
| Gateway                            | API를 요청하기 위한 게이트 웨이로서                              | TwitterTimelineGateway, QiitaAccountGateway       |
| Store, Storage, Registry           | DB를 요청하거나 Disk I/O에 의한 데이터 영속화를 하는 부분에 이용 | FavoriteSettingStore, DataStorage, ConfigRegistry |
| Cache                              | -                                                                | TimelineCache                                     |
| Log                                | 로깅 혹은 사용 이력 저장 위치                                    | UsageLog                                          |
| History                            | 이력 저장 위치                                                   | UsageHistory                                      |
| Configuration, Preference, Setting | 설정 데이터 저장 위치                                            | TimelineConfiguration                             |  
    
	
| 이름             | 보충                                 | 예                         |
|------------------|--------------------------------------|----------------------------|
| Logger           | 로깅을 실행하다                      | UsageLogger                |
| Cleaner, Sweeper | 데이터를 깨끗이 지우는 역할을 맡는다 | CacheCleaner, CacheSweeper |
  
    
## 데이터를 가공하는 레이어
  
| 이름      | 보충                                          | 예                     |
|-----------|-----------------------------------------------|------------------------|
| Filter    | 데이터를 엄선하다                             | TimelineFilter         |
| Extractor | 어느 데이터에서 다른 데이터를 추출하기        | MessageExtractor       |
| Formatter | 어느 데이터를 성형하여 다른 데이터를 출력한다 | MessageFormatter       |
| Collector | 데이터를 가져온다                             | AnalyticsDataCollector |
  
  
## 데이터 소스를 랩핑하는 레이어
  
| 이름         | 보충                                                                                                           | 예                       |
|--------------|----------------------------------------------------------------------------------------------------------------|--------------------------|
| Provider     | 데이터의 제공자라는 뜻으로 DB, http 통신, cache 등을 캡슐화한 상위 레이어의 것. 혹은 Android의 ContentProvider | TwitterTimelineProvider  |
| Manager      | 데이터를 관리한다. 이른바 모델의 흔한 이름                                                                     | AccountManager           |
| Loader       | 데이터 읽기. 혹은 Android의 Loader                                                                             | TimelineLoader           |
| Logger       | 로그에 쓰기 시작, 혹은 Log로의 액세스를 제공하는 추상 계층                                                     | RecentUsageLogger        |
| Configurator | 설정의 초기 값을 저장하거나 어떤 데이터를 바탕으로 자동으로 설정을 저장하는 클래스                             | FirstSettingConfigurator |
| Migrator     | 버전 업 등에서 데이터 구조에 변경이 있을 때의 논리를 맡는다                                                    | UserDataMigrator         |
  
  
## 비동기 처리를 다루는 계층
   
| 이름                            | 보충                                                                                                  | 예                                     |
|---------------------------------|-------------------------------------------------------------------------------------------------------|----------------------------------------|
| Job, Task, Runnable, Executable | 비동기 처리의 덩어리를 다룬다                                                                         | UploadJob, MigrationTask               |
| Runner, Executor, Worker        | 주어진 Job 이나 Task를 실행하다                                                                       | UploadJobRunner, MigrationTaskExecutor |
| Aware                           | 동기 처리에서 어떠한 콘텍스트를 보유하고 관리되는 것을 나타내는 인터페이스(Spring Framework에서 이용) | ApplicationContextAware                |
  
  
## 프레임워크로의 접근을 통합하는 레이어
  
| 이름     | 보충                                                                      | 예                           |
|----------|---------------------------------------------------------------------------|------------------------------|
| Facade   | Facade 패턴 구현                                                          | BoundServiceFacade           |
| Service  | 호환성을 위해 구현을 캡슐화하여 각각의 기능에 대한 접속을 실현하는 레이어 | ApplicationControllerService |
| Resolver | 사용자 환경에 따른 처리 라우팅을 하는 레이어                              | ContentResolver              |
  
  
## UI 상의 동작을 내포하는 클래스
  
| 이름                | 보충                                       | 예                                        |
|---------------------|--------------------------------------------|-------------------------------------------|
| Action              | 조작 자체를 나타내다                       | SubmitAction, CancelAction                |
| Dispatcher, Handler | 조작을 받아서 처리를 수행한다              | SubmitActionDispatcher, UserActionHandler |
| Listener, Watcher   | 조작을 감시한다. Observer 유형의 구현 이름 | ClickListener, TextEditWatcher            |
    
  
<br>  
<br>  
  
출처: https://qiita.com/KeithYokoma/items/ee21fec6a3ebb5d1e9a8  
