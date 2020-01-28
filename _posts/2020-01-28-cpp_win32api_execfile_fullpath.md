---
layout: post
title: C++ - Win32API 실행 파일의 full path 얻기
published: true
categories: [C++]
tags: c++ win32api getApplicationFilePath
---
Win32 API의 'GetModuleFileName' 함수 사용.  
  
```
std::wstring getApplicationFilePath() 
{
    wchar_t fileName[MAX_PATH] = { 0 };
    ::GetModuleFileNameW(nullptr, fileName, MAX_PATH - 1);
    return fileName;
}
``` 
  
<br>  
  
## 파일의 full path에서 실행 파일 이름만 추출하기  
- getApplicationFileName()
- _wsplitpath_s 함수로 디렉토리와 파일 이름을 나눈다.
    - https://technet.microsoft.com/ko-kr/library/8e46eyt7(v=vs.110).aspx  
  
```
std::wstring getApplicationFileName() {
    wchar_t drive[MAX_PATH];
    wchar_t dir[MAX_PATH];
    wchar_t filename[MAX_PATH];
    wchar_t ext[MAX_PATH];
    wchar_t exe_filename[MAX_PATH];

    auto filePath = getApplicationFilePath();
    ::_wsplitpath_s(filePath.c_str(),
        drive,
        MAX_PATH,
        dir,
        MAX_PATH,
        filename,
        MAX_PATH,
        ext,
        MAX_PATH);
    ::_wmakepath_s(exe_filename, MAX_PATH, nullptr, nullptr, filename, ext);
    return filename;
}
```
  
_makepath_s : full path 문자열을 만든다      
```
_makepath_s( path_buffer, _MAX_PATH, "c", "\\sample\\crt\\",  
                      "crt_makepath_s", "c" );  
```  
결과  
<pre> 
c:\sample\crt\crt_makepath_s.c
</pre>
  
<br>
  
_splitpath_s : full path 문자열에서 드라이브, 디렉토리, 파일이름, 확장자를 추출한다.  
```
_splitpath_s( path_buffer, drive, _MAX_DRIVE, dir, _MAX_DIR, fname,  
                       _MAX_FNAME, ext, _MAX_EXT );  
```  
결과  
<pre> 
Drive: c:  
Dir: \sample\crt\  
Filename: crt_makepath_s  
Ext: .c  
</pre>   
  