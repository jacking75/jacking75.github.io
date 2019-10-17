---
layout: post
title: 최신 Windows SDK의 IOCP GetQueuedCompletionStatus, CreateIoCompletionPort 함수의 인자 타입 변경점
published: true
categories: [Network]
tags: iocp windows network c++ 네트워크 CreateIoCompletionPort GetQueuedCompletionStatus
---
## GetQueuedCompletionStatus
이전 사용 코드  
```
//CompletionKey를 받을 포인터 변수
stClientInfo* pClientInfo = NULL;
//함수 호출 성공 여부
BOOL bSuccess = TRUE;
//Overlapped I/O작업에서 전송된 데이터 크기
DWORD dwIoSize = 0;
//I/O 작업을 위해 요청한 Overlapped 구조체를 받을 포인터
LPOVERLAPPED lpOverlapped = NULL;

while( m_bWorkerRun )
{
    //////////////////////////////////////////////////////
    //이 함수로 인해 쓰레드들은 WaitingThread Queue에
    //대기 상태로 들어가게 된다.
    //완료된 Overlapped I/O작업이 발생하면 IOCP Queue에서
    //완료된 작업을 가져와 뒤 처리를 한다.
    //그리고 PostQueuedCompletionStatus()함수에의해 사용자
    //메세지가 도착되면 쓰레드를 종료한다.
    //////////////////////////////////////////////////////
    bSuccess = GetQueuedCompletionStatus( m_hIOCP,
        &dwIoSize,                    // 실제로 전송된 바이트
        (LPDWORD)&pClientInfo,        // CompletionKey
        &lpOverlapped,                // Overlapped IO 객체
        INFINITE );                    // 대기할 시간
            
```  
위 코드를 컴파일 하면 아래와 같은 에러가 나온다  
> 심각도    코드    설명    프로젝트    파일    줄    비표시 오류(Suppression) 상태 오류    C2664    'BOOL GetQueuedCompletionStatus(HANDLE,LPDWORD,PULONG_PTR,LPOVERLAPPED *,DWORD)': 인수 3을(를) 'LPDWORD'에서 'PULONG_PTR'(으)로 변환할 수 없습니다.    02_IOCP    e:\github\codes_book_onlinegameserver\02_iocp\iocompletionport.cpp    356    
  
아래처럼 수정해야 한다.    
```
bSuccess = GetQueuedCompletionStatus( m_hIOCP,
            &dwIoSize,                    // 실제로 전송된 바이트
            (PULONG_PTR)&pClientInfo,        // CompletionKey
            &lpOverlapped,                // Overlapped IO 객체
            INFINITE );                    // 대기할 시간
```            
  
  
  
## CreateIoCompletionPort
이전 사용 코드  
```
HANDLE hIOCP;
//socket과 pClientInfo를 CompletionPort객체와 연결시킨다.
hIOCP = CreateIoCompletionPort((HANDLE)pClientInfo->m_socketClient
    , m_hIOCP
    , reinterpret_cast<DWORD>( pClientInfo ) , 0);
```    
  
수정된 코드  
```
hIOCP = CreateIoCompletionPort((HANDLE)pClientInfo->m_socketClient
        , m_hIOCP
        , (ULONG_PTR)( pClientInfo ) , 0);
if( NULL == hIOCP  || m_hIOCP != hIOCP )
{
    printf("[에러] CreateIoCompletionPort()함수 실패: %d",GetLastError() );
    return false;
}
```
  
  
  
[MS Docs의 CreateIoCompletionPort](https://docs.microsoft.com/ko-kr/windows/win32/fileio/createiocompletionport/?WT.mc_id=DT-MVP-4024485 )  
[MS Docs의 GetQueuedCompletionStatus](https://docs.microsoft.com/ko-kr/windows/win32/api/ioapiset/nf-ioapiset-getqueuedcompletionstatus/?WT.mc_id=DT-MVP-4024485 )  
 GetQueuedCompletionStatus 함수의 새 버전은 [GetQueuedCompletionStatusEx](https://docs.microsoft.com/ko-kr/windows/win32/fileio/getqueuedcompletionstatusex-func/?WT.mc_id=DT-MVP-4024485 ) 이다.  