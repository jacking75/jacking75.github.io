---
layout: post
title: C++ - 배치파일을 사용하여 원격 디버깅 할 때 IP 주소 변경 및 설치, 설정 등등 한방에 하기
published: true
categories: [C++]
tags: c++ vc debug remote windows
---
[출처](https://qiita.com/tera1707/items/4483b67bae0db0f9c783 )  
  
## 방법
아래와 같은 배치 파일을 만든다. 하고있는 것은 아래와 같다.  
  
1. IP 주소를 고정으로 세팅(여기에서는 192.168.1.111 로 하고 있다)
2. 폴더 공유를 활성화 한다
3. "응용 프로그램 및 안전하지 않은 파일의 시작" 설정을 변경 (경고 창을 내지 않는다)
4. 바탕 화면에 "test" 폴더를 만들고, 공유한다(여기를 작업용 PC와 파일 교환 폴더로 하는 의도)
5. 원격 데스크톱을 유효하게 한다
6. 파일 폴더의 공유를 활성화 한다(VisualStudio에서 원격 PC에 출력 할 수 있도록)
  
  
## 의도
바탕 화면에 폴더(test 폴더)을 만들고 자신의 작업 PC의 Visualstudio에서 빌드한 것을 여기에 출력되도록 설정하고 사용한다.  
  
응용 프로그램 및 안전하지 않은 파일 실행 설정은 원격에서 실행한 응용 프로그램에서 다른 exe를 실행하려고 할 때 경고 창이 나오는 것을 방지한다.  
  
  
리포트 디버그 하고 싶은 컴퓨터의 일괄 설정.bat  
```
@echo off

rem -------------------------------------------------------
rem IP 주소를 고정으로 하는 설정
rem ※ 영어, 한국어 양쪽으로 시도한다
rem -------------------------------------------------------
netsh interface ip set address "Ehternet 2" static 192.168.1.111 255.255.255.0
netsh interface ip set address "이더넷 2" static 192.168.1.111 255.255.255.0
echo;

rem -------------------------------------------------------
rem 「응용 프로그램 및 안전하지 않은 파일 실행 」을 편진
rem 0:활성화 한다 ★
rem 1:다이얼로그 표시
rem 3:비 활성화 한다 
rem -------------------------------------------------------
Reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\3" /v "1806" /t REG_DWORD /d "0" /f

rem -------------------------------------------------------
rem 바탕화면에 「test」 폴더를 만들고, 공유한다
rem -------------------------------------------------------
set DESKTOPPATH=%USERPROFILE%\Desktop
set SHAREFOLDERPATH=%DESKTOPPATH%\test
cd %DESKTOPPATH%

mkdir test
net share test /delete
net share test="%SHAREFOLDERPATH%" /grant:everyone,full

rem -------------------------------------------------------
rem 리모트 데스크탑을 허용하는 설정 실시
rem 0:허가 한다
rem 1:불허 한다
rem -------------------------------------------------------
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d "0" /f

rem -------------------------------------------------------
rem 폴더 공유를 활성화 한다
rem 여기는 자동화 할 수 없으므로 수동을 더한다
rem -------------------------------------------------------
control.exe /name Microsoft.NetworkAndSharingCenter
echo 왼쪽에 있는 「공유 상세 설정 변경」을 누른다
echo 「공유 상세 설정 변경」 화면이 열린다
echo;
pause
echo;
echo 1."개인"에서 "파일 및 프린터 공유"에서 "파일 및 프린터 공유를 활성화 한다"에 체크한다.
pause
echo;
echo 2."게스트 또는 공용"에서 "파일 및 프린터 공유"에서 "파일 및 프린터 공유를 활성화 한다"에 체크한다.
pause
echo;
echo 3."도메인"에서 "파일 및 프린터 공유"에서 "파일 및 프린터 공유를 활성화 한다"에 체크한다. 
pause
echo;
echo 4."모든 네트워크"에서 "공용 폴더 공유"에서 "공유를 활성화하고 ..."에 체크한다. 
pause
echo 수동 설정 끝
echo;

rem -------------------------------------------------------
rem 리모트 디버거를 설치한다
rem -------------------------------------------------------
echo 리모트 디버거를 설치한다
%~dp0vs_remotetools.exe

rem -------------------------------------------------------
rem 끝
rem -------------------------------------------------------
echo 작업 완료

pause
```
  
※ 동일한 계층에 MS의 페이지에서 다운로드 한 원격 디버거 설치(vs_remotetools.exe)를 놓아두어야 한다.  
→ 원격 디버거 입수 등은 [여기를 참조](https://qiita.com/tera1707/items/5d70ebf96d03c1042b9e ).  
  
  
  
  