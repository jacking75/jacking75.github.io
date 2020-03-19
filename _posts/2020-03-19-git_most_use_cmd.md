---
layout: post
title: git - 자주 사용하는 git 명령어
published: true
categories: [Git]
tags: git
---
[출처](https://qiita.com/taisukek/items/1c4fbb4ea4f7c2d4fad1 )  
  
## 자신의 commit을 pull request 할 때까지
  
```
git pull #로컬 저장소를 최신으로
git checkout -b <branchName> #브랜치 작성과 이동
#파일 편진
git status #편집한 파일을 확인
git diff #차분을 확인
git add . #"."은 현재 디렉토리 이하의 파일을 모두 add
#git add -p 파일이름 #개변 범위가 많은 경우는 p 옵션으로 블럭 단위로 스테이징
git commit -v
git push -u origin <newRemoteBranchName>
#GUI로 파일 차분을 한번 더 확인하고 풀 리퀘스트
```
  
  
## 리모트의 Branch를 로컬에 check out 하기
  
```
git checkout -b <newLocalBranchName> <remoteBranchName>
```
  
  
## add를 잘 못했을 때
  
```
git status #add한 파일을 확인
git reset HEAD <fileName>
```
    
    
  
## commit을 잘 못했을 때
  
### commit을 회수
파일 변경을 되 돌린다  
```
git status #commit을 확인

git reset --hard HEAD^
```
  
파일 변경을 되 돌리지 않는다  
```
git status #commit을 확인
git reset --soft HEAD^
```
  
  
### 커밋을 덮어 쓴다
  
```
git status #commit을 확인
#파일 편진
git add <filename>
git commit --amend #push 끝난 commit에는 실시하지 않는다
```
  
  
  
## push를 잘 못 했을 때
  
### 자신만 작업한 Branch의 경우
  
```
git reset --hard HEAD^
git push -f #commit을 강제적으로 덮어 쓰고 있으므로 다른 사람도 작업하고 있는 경우 commit 이력이 날라 갈 수 있다
```
  
### 타인도 같이 작업한 Branch의 경우
  
```
git log #commit id를 확인
git revert <commitId> #취소 commit을 만든다
```
  
  
  
## pull request가 충돌하였다
  
```
git checkout <yourWorkingBranchName> #작업 중의 Branch로 이동
git pull #로컬 저장소 최신으로
git rebase master #마스터 브랜치의 최신 commit에, 작업 브랜치의 파생 commit을 옮겨 심는다
git status #충돌 상황을 확인
#파일을 편집
git add <fixedFileName> #경합 파일을 편집
git rebase --continue #rebase를 진행한다. 명령어 지시에 따라서 add와 rebase를 반복한다
git push #remote에 push 한다
```  
   
   
   
## Branch에서 작업 중에 다른 Branch에서의 작업을 요청 받았다
  
```
git stash save <yourComment> #변경 내용을 임시 보존
git stash list #보존된 것을 확인
git checkout <otherBranchName> #다른 Branch로 이동
```
  
  
  
## 다른 브랜치에서 작업 완료
  
```
git checkout <originalBranchName> #원래의 Branch로 돌아온다
git stash pop #변경 내용을 되돌린다.
```
  
  
