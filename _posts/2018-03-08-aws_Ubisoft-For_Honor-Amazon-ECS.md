---
layout: post
title: Ubisoft 사례 - For Honor에서의 Amazon ECS 사용법
published: true
categories: [Network]
tags: aws ubi 번역
---
AWS re:Invent 2017에서의 Ubisoft 세션을 정리한 글.    
[원문(일본어)](https://dev.classmethod.jp/cloud/aws/reinvent2017-gam307-ubisoft-ecs/)  
[AWS ECS 사전 지식](http://yongho1037.tistory.com/732)  
    
<br>  
   
###  강연자
Ralf Mueller - Online Technical Architect, Ubisoft  
Louis-Michel Gélinas - DevOps Team Lead, Ubisoft  
  
### 세션 내용
#### Ubisoft
올해 20 주년. 몬테리올의 회사.  
신작 게임 제목 For Honor.  
오픈 β에서는 6 빌리언 유저가 참여.  
  
#### Fail fast! Succeed consistently!
게임 개발에 대해서는 실패할 수 있다.  
실패하면 롤백하여 성공의 길을 찾는다.
처음부터 성공의 길을 걷는 경우는 적다.  
  
#### For Honor의 프로덕션 환경
ECS는 Front-end와 Backend 양쪽에서 작성.
전장은 S3에서 CloudFront로 전달.  
![](https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2017/11/DSC01129-960x640.jpg)   
  
#### 초기 서비스 개발
애플리케이션은 Play에서 개발.  
Couchbase에 데이터를 보관.  
AWS Elastic Beanstalk로 배포.  
  
#### 막 다른 골목
Play는 잘 맞지 않았다.  
Elastic Beanstalk의 관리가 힘들다.  
Couchbase도 잘 맞지 않았다.  
  
#### 연도의 검토
1년 반 후에는 게임 발매일.  
2개의 미션 크리티컬 하지 않는 서비스.  
    파벌 싸움.  
    플레이어 정보 리치화.  
Immutable한 Docker image에서 개발하고 싶다.  
개발 환경은 최소한으로.  
프로덕션 환경은 스케일 아웃 가능하게.  
  
#### 개발 변경
인프라 기반은 CloudFormation 템플릿화.  
ECS 태스크나 서비스 관리는 Python으로 개발.  
aws-cli에서 사람이 에뮬레트.  
발생하는 문제.  
    CloudFormation에서는 미세 조정을 추적할 수 없다.  
    Python 스크립트에서는 오케스트레이션이 힘들다.  
  	
#### 라스트 마일
Datadog에서도 모니터링 및 경보 발포.  
Log나 매트릭스를 비쥬얼라이즈.  
KPI를 트레킹.  
  
#### Trail에서 Auto ban까지
ECS에서 Front-end와 Backend의 오토 스케일링 그룹을 세운다.  
ECS의 boot 시퀀스.  
    aws-cli를 설치.  
    OpsWorks에 인스턴스를 등록.  
    ECS 에이전트를 셋업.  
유저 수용 테스트 환경과 프로덕션 환경은 노트 이퀄.  
   유저 수용 테스트 환경은 최소한의 심플함.  
Security Update의 대응은 SNS와 Lambda을 조합하여 실시.  
    AWS Compute Blog를 참고로.  
    	 
#### CloudFormation stack의 관리
Git으로 템플릿 관리.   
Gitlab의 CI job을 트리거로 실시.  
ECS의 서비스와 태스크를 CloudFormation stack에서 작성.  
    Python 코드를 템플릿으로 변환.  
  	
#### 유저 수용 여건과 접속
유저 수용 환경에 필요한 온프레미스 환경은 인터넷에 접속되지 않는다.
VPN 터널에서 AWS 환경과 접속.  
  
#### 배운 것
모든 수동 조작은 리스크.  
모든 것을 검증한다. CloudFormation은 도움이 된다.  
서비스 컨테이너 관리는 힘들다.  
Lambda는 덕 테이프처럼 사용할 수 있고, 많은 기능을 싸게 구현 가능.  
관리 서비스 최고.  
  
#### 마지막으로
ECS는 꽤 익수하다고 생각하지만, 그래도 서비스 컨테이너의 관리에는 고생했던 것 같다.  
  