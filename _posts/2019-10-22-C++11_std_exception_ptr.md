---
layout: post
title: C++11 - std::exception_ptr 사용하기
published: true
categories: [C++]
tags: c++ c++11 std::exception_ptr
---
[출처](https://qiita.com/LNSEAB/items/c76ed1a4e4d69b824655)  
  
Windows의 윈도우 관련 프로그래밍을 할 때 반드시 나오는 윈도우 프로시저이지만, 윈도우 프로시저 외부에 그대로 예외를 던지려하면 무시되거나 프로세스를 강제 종료하는 것이 MSDN의 WindowProc callback function의 Remarks에 적혀 있다.  
[WindowProc callback function](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633573.aspx  )  
  
그래서 C++11에서 추가된 std::exception_ptr에 윈도우 프로시저의 예외를 일단 저장하고, 메시지 루프에서 던지는 것으로 문제 없이 예외 처리를 할 수 있도록 한다.   
그럼 std::exception_ptr을 사용한 윈도우 프로시저 및 메시지 루프를 다음에 나타낸다.  
  
    
### 윈도우 프로시저
윈도우 프로시저 내에서 발생한 모든 예외를 std::current_exception를 사용하여 std::exception_ptr에 일단 저장하고, 윈도우 프로시저에서 그대로 예외가 밖으로 나오지 않도록 한다.  
```
// 윈도우 프로시져과 메시지 루프가 있는 함수에서 보이도록
// 일단 글로벌 변수로 하는 proc_except
std::exception_ptr proc_except;

LRESULT CALLBACK procedure(HWND hwnd, UINT msg, WPARAM wparam, LPARAM lparam) noexcept
{
    try {
        if( msg == WM_DESTROY ) {
            PostQuitMessage( 0 );
            return 0;
        }

        // 여기에서 예외를 던져 보는 테스트
        throw std::runtime_error( "error" );
    }
    catch( ... ) {
        // 예외를 전부 잡고, 예외를 exception_ptr 에 저장한다
        proc_except = std::current_exception();
        // 일단 프로시져에서 던진다
        return 0;
    }

    return DefWindowProcW( hwnd, msg, wparam, lparam );
}
```  
  
### 메시지 루프
윈도우 프로시저에서 던져진 예외를 메시지 루프에서 받고, std::exception_ptr 에 예외가 포함 되어 있는 경우 std::rethrow_exception으로 메시지 루프에서 예외를 다시 던진다. 나머지는 보통의 예외 처리처럼 try-catch로 처리 할 수 ​​있다.  
```
// 글로벌 변수
// std::exception_ptr proc_except;

MSG msg;
for(;;) {
    auto const ret = GetMessageW( &msg, nullptr, 0, 0 );
    if( ret == 0 || ret == -1 ) {
        break;
    }

    DispatchMessageW( &msg );


    // 프로시져에서 발생한 예외가 있다면 다시 던진다
    if( proc_except ) {
        std::rethrow_exception( proc_except );
    }
}
```  
  
  
### WM_CREATE 에서 예외를 던질 경우
WM_CREATE에서 예외를 던지는 경우는 CreateWindowEx 나 CreateWindow 직후에 std::exception_ptr 예외가 저장되어 있는지를 알아보고, std::rethrow_exception를 호출한다. 또한 CreateWindowEx 자체가 실패하는 것을 생각하여 std::exception_ptr을 체크하는 것과는 별도로 반환 값을 확인하는 것이 좋다.  
```
HWND hwnd = CreateWindowExW( 
    0, wc.lpszClassName, L"window", WS_OVERLAPPEDWINDOW,
    0, 0, 1024, 769, nullptr, nullptr, wc.hInstance, nullptr
);
// 프로시져에서 던져진 예외는 이쪽
if( proc_except ) {
    std::rethrow_exception( proc_except );
}
// CreateWindowEx 자체가 실패한 경우는 NULL을 반환하는 것으로
if( !hwnd ) {
    throw std::runtime_error( "CreateWindowEx failed" );
}
```  
  
덧붙여서, 윈도우 프로시저의 WM_CREATE에서 예외를 발생 시켰을 때, catch에서 -1을 돌려주는 것을 생각하여 std::exception_ptr, hwnd의 NULL 체크 순서로 되어 있다.   이것은 WM_CREATE의 처리에 -1을 돌려 주면 CreateWindowEx가 NULL을 반환하기 때문이다.  
  
  
### 코드  
메시지 루프에서 std::runtime_error가 발생하여 error를 에러 출력하도록 하고 있다.  
```
#include <windows.h>
#include <exception>
#include <iostream>

std::exception_ptr proc_except;

LRESULT CALLBACK procedure(HWND hwnd, UINT msg, WPARAM wparam, LPARAM lparam) noexcept
{
    try {
        if( msg == WM_DESTROY ) {
            PostQuitMessage( 0 );
            return 0;
        }

        // 여기에서 예외를 던져 본다
        throw std::runtime_error( "error" );
    }
    catch( ... ) {
        proc_except = std::current_exception();
        return 0;
    }

    return DefWindowProcW( hwnd, msg, wparam, lparam );
}

int main()
{
    try {
        WNDCLASSEXW const wc = {
            sizeof( WNDCLASSEXW ), CS_VREDRAW | CS_HREDRAW, procedure,
            0, 0, GetModuleHandleW( nullptr ), nullptr, LoadCursorA( nullptr, IDC_ARROW ),
            static_cast< HBRUSH >( GetStockObject( WHITE_BRUSH ) ),
            nullptr, L"nanka_window_class_name", nullptr
        };
        if( !RegisterClassExW( &wc ) ) {
            throw std::runtime_error( "RegisterClassEx failed" );
        }

        HWND hwnd = CreateWindowExW( 
            0, wc.lpszClassName, L"window", WS_OVERLAPPEDWINDOW,
            0, 0, 1024, 769, nullptr, nullptr, wc.hInstance, nullptr
        );
        // 프로시져에서 던져진 예외는 이쪽
        if( proc_except ) {
            std::rethrow_exception( proc_except );
        }
        // CreateWindowEx 자체가 실패한 경우는 NULL 이 반환되므로
        if( !hwnd ) {
            throw std::runtime_error( "CreateWindowEx failed" );
        }

        ShowWindow( hwnd, SW_SHOW );
        UpdateWindow( hwnd );

        MSG msg;
        for(;;) {
            auto const ret = GetMessageW( &msg, nullptr, 0, 0 );
            if( ret == 0 || ret == -1 ) {
                break;
            }

            DispatchMessageW( &msg );

            // 프로시져에서 발생한 예외가 있다면 다시 던진다
            if( proc_except ) {
                std::rethrow_exception( proc_except );
            }
        }
    }
    catch( std::exception const& e ) {
        std::cerr << e.what() << std::endl;
    }
    catch( ... ) {
        std::cerr << "unknown error" << std::endl;
    }
}
```  
  
