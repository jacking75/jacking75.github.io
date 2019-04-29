---
layout: post
title: premake
published: true
categories: [C++]
tags: c++ premake vc make
---
- CMake와 같은 멀티 플랫폼 오픈소스 개발에 유용한 툴.
- lua를 통해 스크립트를 작성한다. 루아 파일 이름과 같은 이름의 솔루션 파일을 만들어준다.
- 현재 premake4와 premake5 두 가지 버전이 있다.
    - Visual Studio 2013 이상을 지원하려면 premake5를 사용해야 한다.
- C, C++, or C# projects targeting:
    - Microsft Visual Studio 2008-2015
    - GNU Make, including Cygwin and MinGW
    - Previous version of Premake also supported exporting for Xcode, MonoDevelop, Code::Blocks, and CodeLite. We are in the process of bringing these exporters back online for the final release.
- Premake 5.0 generated projects can support:
    - 32- and 64-bit builds
    - Xbox 360 (Visual Studio only)
- [Addon module](https://github.com/premake/premake-core/wiki/Modules)
- [오피셜 페이지](https://premake.github.io/)
- [깃허브 페이지](https://github.com/premake/premake-core)
- [Premake를 사용하는 단체 혹은 프로젝트](https://github.com/premake/premake-core/wiki/Who-Uses-Premake)
- [premake-cookbook(ver 4 기준)](https://github.com/premake/premake-cookbook)
  
  
  
## 키워드
- location  
    - 쓰기 폴더(작성 환경에서의 패스가 된다)  
    - 후에 filter에서 대응 플랫폼에 나누어 쓸 수 있다.
- characterset  
    - MBCS는 멀티 바이트 문자 세트  
- kind  
    - StaticLib: .lib  
    - SharedLib: .dll  
    - ConsoleApp: 콘솔 응용 프로그램  
    - WindowedApp: win32 어플리케이션  
- postbuildcommands  
    - 빌드 전에 실행하는 명령어  
    - fxc 등을 실행하는 것도 가능  
- filter  
    - 특정 환경에서의 동작을 지정  
- flags  
    - Symbols:심볼 정보를 읽기(브레이크 포인트 정보)  
  
다른 키워드는 아래 링크의 코드를 참고한다.  
https://github.com/GPUOpen-LibrariesAndSDKs/ForwardPlus11  
  
  
  
## 실습
### hello world
  
<pre>
+ HelloWorld
  + hello.c
  + premake5.lua
  + premake5.exe
  + vc2013.bat
</pre>
  
```
//hello.c
#include <stdio.h>

int main(void) {
   puts("Hello, word!");
   return 0;
}
```
  
```
-- premake5.lua
solution "HelloWorld"
do
    configurations { "Debug", "Release" }
end

project "HelloWorld"
do
    kind "ConsoleApp"
    language "C"
    -- todo: この記述の意味
    targetdir "bin/%{cfg.buildcfg}"

    files { "**.h", "**.c" }

    filter "configurations:Debug"
    do
        defines { "DEBUG" }
        flags { "Symbols" }
    end

    filter "configurations:Release"
    do
        defines { "NDEBUG" }
        optimize "On"
    end
end
```
  
  
### simple_console. 간단한 기능의 console 프로젝트
  
```
-- A solution contains projects, and defines the available configurations
solution "MyApplication"
  configurations { "Debug", "Release" }

  -- A project defines one build target
  project "MyApplication"
    kind "ConsoleApp"
    language "C++"
    files { "**.h", "**.cpp" }

    configuration "Debug"
      defines { "DEBUG" }
      flags { "Symbols" }

    configuration "Release"
      defines { "NDEBUG" }
      flags { "Optimize" }
```
  
빌드 전  
![](/images/2019/premake_img001.png)  
빌드 후  
![](/images/2019/premake_img002.png)  
VS 상태  
![](/images/2019/premake_img003.png)  
  
  
### 출력 디렉토리를 분리한다
- build 디렉토리에 파일이 생성되도록 한다
  
```
-- premake5.lua
location "build"

solution "HelloWorld"
do
    configurations { "Debug", "Release" }
end

project "HelloWorld"
do
    kind "ConsoleApp"
    --language "C"
    language "C++"
    targetdir "build/%{cfg.buildcfg}"

    files { "**.h", "**.cpp" }

    filter "configurations:Debug"
    do
        defines { "DEBUG" }
        flags { "Symbols" }
    end

    filter "configurations:Release"
    do
        defines { "NDEBUG" }
        optimize "On"
    end
end
```
  
<pre>
+ HelloWorld
  + .gitignore
  + hello.c
  + premake5.lua
  + premake5.exe
  + vc2013.bat
  + build
     + HelloWorld.sln
     + HelloWorld.vcxproj
     + Debug
        + HelloWorld.exe
     + Release
        + HelloWorld.exe
</pre>
  
- make의 clean과 비슷한 방법으로 premake5 clean 을 사용하면 build 디렉토리가 삭제된다
  
  
### 실용적인 예
- sln, vcxproj 등의 출력 디렉터리를_build_premake에 격리
- 32bit, 64bit의 2종류
- Debug, Release의 2종류
- 성과물(exe, dll)의 출력을_build_premake/Win64_Release 라는 식으로 지정
- 출력 디렉터리를 나눈 것으로 라이브러리 이름에_d 같은 서픽스는 붙이지 않는다
  
premake5.lua    
```
local build_dir="_build_premake"

-- premake5.lua
location(build_dir)

workspace "workspace"
do
    configurations { "Debug", "Release" }
    platforms { "Win32", "Win64" }
end

filter "configurations:Debug"
do
    defines { "DEBUG", "_DEBUG" }
    flags { "Symbols" }
end

filter "configurations:Release"
do
    defines { "NDEBUG" }
    optimize "On"
end

filter { "platforms:Win32" }
   architecture "x86"
filter {"platforms:Win32", "configurations:Debug" }
    targetdir(build_dir.."/Win32_Debug")
filter {"platforms:Win32", "configurations:Release" }
    targetdir(build_dir.."/Win32_Release")

filter { "platforms:Win64" }
   architecture "x64"
filter {"platforms:Win64", "configurations:Debug" }
    targetdir(build_dir.."/Win64_Debug")
filter {"platforms:Win64", "configurations:Release" }
    targetdir(build_dir.."/Win64_Release")

filter { "action:vs*" }
    buildoptions {
        "/wd4996",
    }
    defines {
        "_CRT_SECURE_NO_DEPRECATE",
        "NOMINMAX",
    }
    --characterset "Unicode"
    characterset "MBCS"

filter {} -- filter clear


project "project"
do
    --kind "ConsoleApp"
    --kind "WindowedApp"
    --kind "StaticLib"
    kind "SharedLib"
    --language "C"
    language "C++"

    objdir "%{prj.name}"

    flags{
        --"WinMain" -- with WindowedApp kind
        --"StaticRuntime",
    }
    files {
        "*.cpp",
        "*.h",
    }
    includedirs { 
    }
    defines { 
    }
    buildoptions { 
    }
    libdirs { 
    }
    links { 
    }

    postbuildcommands{
        "copy $(TargetDir)$(TargetName).dll ..",
    }
    filter { "configurations:Debug" }
    do
        postbuildcommands{
            "copy $(TargetDir)$(TargetName).pdb ..",
        }
    end
end
```
  
  
### 솔루션과 프로젝트를 나눈다

```
-- premake5.lua
location "build"

solution "HelloWorld"
do
    configurations { "Debug", "Release" }
end

-- 서브 디렉토리의 premake5.lua 에 기술된 Project를 읽는다
include "src"
```
  
```
-- src/premake5.lua
project "HelloWorld"
do
    kind "ConsoleApp"
    --language "C"
    language "C++"
    targetdir "build/%{cfg.buildcfg}"

    files { "**.h", "**.cpp" }

    filter "configurations:Debug"
    do
        defines { "DEBUG" }
        flags { "Symbols" }
    end

    filter "configurations:Release"
    do
        defines { "NDEBUG" }
        optimize "On"
    end
end
```
  
<pre>
+ HelloWorld
  + .gitignore
  + src
      + premake5.lua
      + hello.cpp
  + premake5.lua
  + premake5.exe
  + vc2013.bat
  + build
     + HelloWorld.sln
     + HelloWorld.vcxproj
     + Debug
        + HelloWorld.exe
     + Release
        + HelloWorld.exe
</pre>        
  
  
### Filter로 Debug, Release와 win32, win64 라는 4개 패턴의 프로젝트
  
```
-- premake5.lua
location "build"

solution "HelloWorld"
do
    configurations { "Debug", "Release" }
    platforms { "Win32", "Win64" }
end

--
filter { "platforms:Win32" }
   architecture "x32"
filter { "platforms:Win64" }
   architecture "x64"

filter {"platforms:Win32", "configurations:Debug" }
    targetdir "build/Win32/Debug"
filter {"platforms:Win32", "configurations:Release" }
    targetdir "build/Win32/Release"
filter {"platforms:Win64", "configurations:Debug" }
    targetdir "build/Win64/Debug"
filter {"platforms:Win64", "configurations:Release" }
    targetdir "build/Win64/Release"

include "src"
</pre>
<pre>
-- src/premake5.lua
-- targetdir이 빠진 부분

project "HelloWorld"
do
    kind "ConsoleApp"
    --language "C"
    language "C++"

    files { "**.h", "**.cpp" }

    filter "configurations:Debug"
    do
        defines { "DEBUG" }
        flags { "Symbols" }
    end

    filter "configurations:Release"
    do
        defines { "NDEBUG" }
        optimize "On"
    end
end
</pre>
<pre>
+ HelloWorld
  + .gitignore
  + src
      + premake5.lua
      + hello.cpp
  + premake5.lua
  + premake5.exe
  + vc2013.bat
  + build
     + HelloWorld.sln
     + HelloWorld.vcxproj
     + Win32
       + Debug
         + HelloWorld.exe
       + Release
         + HelloWorld.exe
     + Win64
       + Debug
         + HelloWorld.exe
       + Release
         + HelloWorld.exe
```
  
  
### 외부 헤더 파일과 lib 파일 참조하는 프로젝트
  
```
-- premake4.lua
solution "SampleSolution"
   configurations { "Debug", "Release" }

   -- A project defines one build target
   project "SampleProject"
      kind "WindowedApp"
      language "C++"
      files { "**.h", "**.cpp" }

      includedirs { -- -I/usr/include/openglに相当
        "/usr/include/opengl"
      }
      libdirs { -- -L/usr/lib/w32apiに相当
        "/usr/lib/w32api"
      }
      links { -- -lopengl32 -lglu32 -lglut32 に相当
        "opengl32", "glu32", "glut32"
      }

      configuration "Debug"
         defines { "DEBUG" } -- -DDEBUG
         flags { "Symbols" }
         targetname "sample_debug"

      configuration "Release"
         defines { "NDEBUG" } -- -NDEBUG
         flags { "Optimize" }
         targetname "sample"
```
   
premake5.lua  
```
solution "TestModule"
    location "../../"
    configurations{ "Debug", "Release" }
    platforms{ "x64", "x86" }


project "TestModule"
    location "../../build"
    configurations{ "Debug", "Release" }
    platforms{ "x64", "x86" }
    kind "SharedLib"
    language "C++"
    targetdir "../../build/VisualEffectsEditor/"
    characterset("MBCS")

    includedirs{
        "../include/",
        "../../common/include"
    }

    files{
        "../src/**.*",
        "../include/**.h",
    }

       postbuildcommands{}

filter { "platforms:x64", "configurations:Debug" }
    targetsuffix "d"
    defines{ "DEBUG" }
    flags{ "Symbols" }

filter { "platforms:x64", "configurations:Release" }
    defines{ "NDEBUG" }
    flags{ "Optimize" }

filter{ "platforms:x86", "configurations:Debug" }
    targetsuffix "d"
    defines{ "DEBUG" }
    flags{ "Symbols" }

filter{ "platforms:x86", "configurations:Release" }
    defines{ "NDEBUG" }
    flags{ "Optimize" }
```
  
  
### libpng를 사용하는 프로젝트
  
<pre>
+ libpng
    + premake5.lua
    + libpng-1.6.16
        + premake5.lua
    + zlib-1.2.8
        + premake5.lua
</pre>
  
```  
-- premake5.lua
location "build"

solution "libpng"
do
    configurations { "Debug", "Release" }
    platforms { "Win32", "Win64" }
end

defines {
    "WIN32",
    "_WINDOWS",
    "_USRDLL",
}
buildoptions { "/wd4996" }

filter "configurations:Debug"
do
    defines { "DEBUG", "_DEBUG" }
    flags { "Symbols" }
end

filter "configurations:Release"
do
    defines { "NDEBUG" }
    optimize "On"
end

filter { "platforms:Win32" }
   architecture "x32"
filter { "platforms:Win64" }
   architecture "x64"

filter {"platforms:Win32", "configurations:Debug" }
    targetdir "build/Win32/Debug"
filter {"platforms:Win32", "configurations:Release" }
    targetdir "build/Win32/Release"
filter {"platforms:Win64", "configurations:Debug" }
    targetdir "build/Win64/Debug"
filter {"platforms:Win64", "configurations:Release" }
    targetdir "build/Win64/Release"

include "libpng-1.6.16"
include "zlib-1.2.8"
```
  
```
-- zlib-1.2.8/premake5.lua
project "zlib"
do
    kind "StaticLib"
    language "C"
    --language "C++"

    files { "*.h", "*.c" }
end
</pre>
<pre>
-- libpng-1.6.16/premake5.lua
project "libpng"
do
    kind "SharedLib"
    language "C"
    --language "C++"

    files {
        "png.c",
        "pngerror.c",
        "pngget.c",
        "pngmem.c",
        "pngpread.c",
        "pngread.c",
        "pngrio.c",
        "pngrtran.c",
        "pngrutil.c",
        "pngset.c",
        "pngtrans.c",
        "pngwio.c",
        "pngwrite.c",
        "pngwtran.c",
        "pngwutil.c",
    }
    includedirs {
        "../zlib-1.2.8",
    }
    -- 기준 디렉토리는 vcxproj 있는 곳
    prebuildcommands {
        "copy ..\\libpng-1.6.16\\scripts\\pnglibconf.h.prebuilt ..\\libpng-1.6.16\\pnglibconf.h",
    }
    links { "zlib" }
end
```
  
  
### /MT(/MTD)
- premake5.lua 에 기술
  
```
-- buildoptions { "/wd4996" } 다음 행쯤에
flags { "StaticRuntime" }
```
  
  
### 서식
  
```
-- premake5.lua
location "build"

solution "sample"
do
    configurations { "Debug", "Release" }
    platforms { "Win32", "Win64" }
end

filter "configurations:Debug"
do
    defines { "DEBUG", "_DEBUG" }
    flags { "Symbols" }
end

filter "configurations:Release"
do
    defines { "NDEBUG" }
    optimize "On"
end

filter { "platforms:Win32" }
   architecture "x32"
filter { "platforms:Win64" }
   architecture "x64"

filter {"platforms:Win32", "configurations:Debug" }
    targetdir "build/Win32/Debug"
filter {"platforms:Win32", "configurations:Release" }
    targetdir "build/Win32/Release"
filter {"platforms:Win64", "configurations:Debug" }
    targetdir "build/Win64/Debug"
filter {"platforms:Win64", "configurations:Release" }
    targetdir "build/Win64/Release"

project "sample"
do
    kind "ConsoleApp"
    --kind "WindowedApp"
    --kind "StaticLib"
    --kind "SharedLib"
    --language "C"
    language "C++"

    flags{
        --"WinMain"
    }
    files { "*.h", "*.cpp" }
    includedirs {}
    defines {
        "WIN32",
        "_WINDOWS",
    }
    buildoptions { "/wd4996" }
    libdirs {}
    links {}
end
```
  
  
### 어느 lua 파일이 있는 경우만 설정을 추가한다
local.lua  
```
local p = premake
p.override(p.vstudio.vc2010.elements, "user", function(base, cfg)
    -- configurations 마다 호출된다.
    local calls = base(cfg)

    -- \ 을 그대로 출력하고 싶으므로 [[ ]] 로 문자열 설정
    p.x([[<RemoteDebuggerCommand>\\192.168.0.1\$(TargetFileName)</RemoteDebuggerCommand>]])
    p.x([[<RemoteDebuggerServerName>192.168.0.1</RemoteDebuggerServerName>]])
    p.x([[<DebuggerFlavor>WindowsRemoteDebugger</DebuggerFlavor>]])

    -- return calls
end)

-- 빌드 후 이벤트로 실행 파일을 복사
filter "configurations:*"
  postbuildcommands { [[xcopy "$(Configuration)" "\\192.168.0.1\" /Y /Q]] }
filter {}
```
  
premake5.lua  
```  
workspace "sample"
  location "build_premake"
  configurations { "Debug", "Release" }

  project "sample"
    kind            "ConsoleApp"
    language        "C++"
    targetdir       "build_premake/%{cfg.buildcfg}"
    files           { "src/**.h", "src/**.cpp", }
    -- 일단 필터 정리
    filter {}

    -- local.lua 파일이 있다면 읽어서 실행
    local fh = io.open("local.lua", "rb")
    if fh then
      fh:close()
      includeexternal("local.lua")
    end
```
  
  
### 외부 프로젝트 추가
premake5.lua  
```
workspace "sample"
  location "build_premake"
  configurations { "Debug", "Release" }

  externalproject "sampleLib"                        -- 외부 프로젝트
    location "../sampleLib/build_premake/"           -- 상대 패스로 .vcxproj가 있는 디렉토리 지정
    kind     "StaticLib"
    language "C++"

  project "sample"
    kind            "ConsoleApp"                     -- .lib은 "StaticLib"、.dll은 "SharedLib"
    language        "C++"
    targetdir       "build_premake/%{cfg.buildcfg}"  -- %{cfg.buildcfg}는 Debug or Release
    defines         { "NOMINMAX" }                   -- 프리프로세서
    disablewarnings { "4100", "4200" }               -- C4100,C4200 경고를 표시하지 않도록

    files           { "src/**.h", "src/**.cpp", }
    includedirs     { "include" }                    -- 추가 인크루드 디렉토리

    --libdirs       { "lib" }                        -- 추가 라이브러리 디렉토리(필요하다면)
    --links         { "hoge.lib" }                   -- 추가 의존 파일(필요하다면)
    links           { "sampleLib" }                  -- 외부 프로젝트와 링크

    filter "configurations:Debug"                    -- 아래의 Debug 설정
      targetsuffix  "_d"                             -- exe 파일의 서픽스(sample_d.exe 처럼 된다)
      defines       { "DEBUG" }
      flags         { "Symbols" }                    -- 심볼 작성 정보
      optimize      "OFF"                            -- 최적화 없음

    filter "configurations:Release"                  -- 아래의 Release 설정
      optimize      "Speed"                          -- 속도 중시로 최적화
    filter {}                                        -- 필터 클리어
```
  
  
### 정적 라이브러리를 만들 때 외부 라이브러리를 포함하고 있다면
premake5.lua  
```  
local p = premake
function myAdditionalStaticDependencies(cfg)
    -- **.lib 같은 것만 추가
    local links = p.config.getlinks(cfg, "system", "fullpath")
    if #links > 0 then
        links = path.translate(table.concat(links, ";"))
        p.x('<AdditionalDependencies>%s;%%(AdditionalDependencies)</AdditionalDependencies>', links)
    end

    local dirs = p.config.getlinks(cfg, "system", "directory")
    if #dirs > 0 then
        dirs = path.translate(table.concat(dirs, ";"))
        p.x('<AdditionalLibraryDirectories>%s;</AdditionalLibraryDirectories>', dirs)
    end

end
p.override(p.vstudio.vc2010.elements, "lib", function(base, cfg, explicit)
    local calls = base(cfg, explicit)
    if cfg.kind == p.STATICLIB then
        table.insert(calls, myAdditionalStaticDependencies)
    end
    return calls
end)

-- 아래가 빌드 정의
workspace "sample"
  location "build_premake"
  configurations { "Debug", "Release" }

  project "sampleLib"
    kind            "StaticLib"                      -- .lib 파일을 만든다
    language        "C++"
    targetdir       "build_premake/%{cfg.buildcfg}"
    files           { "src/**.h", "src/**.cpp", }

    libdirs         { "lib" }                        -- 추가 라이브러리 디렉토리
    links           { "additional.lib" }             -- 라이브러리 > 추가 의존 파일에 additional.lib 이 추가된다
```  
  
  
### WPF 프로젝트
premake5.lua  
```
solution "TestEditor"
    configurations{ "Debug", "Release" }
    platforms{ "x64", "x86" }
    startproject "TestEditor"
    location "../../"

project "TestEditor"
    platforms{"x64", "x86"}
    location "../../build"
    kind "WindowedApp"
    language "C#"
    flags {"WPF"}
    icon ("../Resource/vfx_icon.ico")
    targetdir "../../build/VisualEffectsEditor/"

    files {
        "../App.config",
        "../**.xaml",
        "../**.cs",
        "../**.ico"
    }

    excludes{
        "../obj/**"
    }

    links ("Microsoft.Csharp")
    links ("PresentationCore")
    links ("PresentationFramework")
    links ("System")
    links ("System.Core")
    links ("System.Data")
    links ("System.Data.DataSetExtensions")
    links ("System.Xaml")
    links ("System.Xml")
    links ("System.Xml.Linq")
    links ("System.Windows.Forms")
    links ("WindowsBase")

    configuration "**.ico"
        buildaction "Resource"

    configuration "Debug"
    do
        flags{ "Symbols" }
    end

    configuration "Release"
    do
        flags{ "Optimize" }
    end

dependson("TestModule")


external("TestModule")
    location "../../build"
    kind "SharedLib"
    language "C++"
    configmap{
        ["Debug"] = "Debug",
        ["Release"] = "Release",
    }
```
  
  
  
## 참고
- [(일어)빌드 premake 소개](http://qiita.com/ousttrue/items/6d837d6daba47b51bd8e)
