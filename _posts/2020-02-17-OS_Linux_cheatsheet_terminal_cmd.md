---
layout: post
title: Linux - 자주 사용하는 명령어
published: true
categories: [OS]
tags: OS Linux terminal cmd
---
[출처](https://qiita.com/taisukek/items/673103c19cec398e81bc )  
  
## 기본
  
```
cd <path> #path로 이동
cd - #이동 전의 path로 이동

pwd #작업 디렉토리의 패스를 표시
ls #현재 디렉토리의 내용 표시

mv <originalPath> <newPath> #파일 이동, 이름 변경
cp <originalPath> <newPath> #파일 복사

find <path> --name "<pattern>" #path 안에서 pattern에 일치하는 파일・디렉토리 표시

rm <filePath> #파일 삭제
rm -r <dirPath> #디렉토리 삭제

tar -zcvf <path>.tar.gz <path> #압축
tar --exclude <pattern> -zcvf <path>.tar.gz <path> #pattern을 제외하고 압축
tar -zxvf <filename> #풀기

history #명령어 입력 이력을 확인
```
  
  
  
## 파일 내용을 표시
  
```  
cat -n <path> #파일 내용을 행수와 표시한다
tail -f <path> #파일 내용을 입력을 기다리리면서 표시(로그 등이 실시간으로 추가되는 것을 확인하고 싶을 때)
vimdiff <path1> <path2> #파일의 차분을 확인한다

vim <path> #파일을 vim 에디터에서 연다
```
  
  
  
## 보완
  
```
<tab> #패스나 명령어 입력 중에 tab을 누르면 보완
<ctrl> + r #입력 후에 문자열을 넣는 것으로 명령어 이력에서 검색해서 보완
```
  
  
  
## 특정 문자열을 포함하는 문서 검색
  
```
grep -R <pattern> <path> #<pattern>을 포함한 행을 path 내의 파일 모두에서 재귀적으로 검색
grep -A <number> #지정한 행수, 히트한 행의 후도 표시(After)
grep -B <number>　#지정한 행수, 히트한 행의 앞도 표시(Before)
grep -i #<pattern>の대문자 소문자를 무시(ignoreCase)
tail -f <fileName> | grep --line-buffered "<pattern>" #tail 되는 내용에 대해서 표시 내용을 검색하고 싶은 경우
```
  
  
  
## 프로세스 관리
  
```
ps -aux | grep <pattern> #패턴이 포함되는 실행 중의 프로세스를 표시

kill <pid> #pid 프로세스를 종료
kill -9 <pid> #pid 프로세스를 강제 종료
```
  
  
  
## 권한 변경
  
```
chmod -R <permission> <path> #path 이하의 폴더・디렉토리의 권한을 재귀적으로 변경
chown -R <user>:<group> <path> #path 아래의 폴더・디렉토리의 user, group을 재귀적으로 변경
```
  