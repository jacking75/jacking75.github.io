---
layout: post
title: League of Legends Platform의 AWS 이행
published: true
categories: [Network]
tags: aws LoL 번역
---
AWS re:Invent 2017에서의 LoL 세션을 정리한 글.    
[원문(일본어)](https://dev.classmethod.jp/cloud/aws/reinvent2017-gam301-game/)
    
<br>  
   
###  강연자
Rob Cameron씨.  
Senior Infrastructure Engineer, Riot Games  
  
### 세션 내용
- 무엇을 해결 했는가?
    - LoL의 컴포넌트는 크게 3가지
        - Platform
            - 로그인/로그아웃
            - 채팅
            - 스토어
            - 매치 메이킹
            - 플레이어 통계
        - rCluster
            - Riot이 자체 개발한 컨테이너 환경
                - Docker + Software Defined Networking
            - 마이크로 서비스 아키텍처
            - All new services
        - Game Server
            - 매우 레이턴시에 엄격
            - 60ms 이하 레이턴시 목표
            - 서버마다 수 백개의 게임이 실행
- Riot shared locations
    - 전 세계에 독자적인 DC를 가지고 있다
        - 기본적으로는 Riot Games의 DC
        - 동남아는 Garena
        - 중국은 Tencent
- 어떻게 해결했는가?
    - AWS API driven architecture
        - 모든 조작이 API로 된다
    - 인프라 관리 툴을 검토
        - 이하의 요건이 필요
            - Configuration as code
            - 외부인이 컨트리 뷰트 할 수 있다
            - 장기간의 서포트
            - 베어 메탈 서포트
            - 모든 주요 퍼블릭 클라우드 서포트
            - 바로 딜리버리 할 수 있다
    - 선정된 툴
        - Packer
            - 이하의 이미지 빌드가 가능
                - AMI
                - VMware OVF
                - Vagrant Virtualbox
				- [MAAS(Metal as a Service)](https://maas.io/) 이미지
			- 잘한 것
			    - Riot사 중에서 채용되었다
			    - 이미지 작성의 요건을 충족하였다
			- 과제
			    - 빌딩 할 때의 파이프 라인을 어떻게 할까?
			    - Windows 이미지 작성
        - Terraform
            - Infrastructure as code
                - 쉽게 편집, 버전 관리가 가능
                - 다른 제품에 접목
                - 하나의 레파지토리에서 많은 배포 가능
             - 잘한 것
                - Riot사 안에서 채용
                - AWS와 MAAS 모두에서 사용할 수 있는
            - 과제
                - 스테이트 관리
                - 오랜 기간 변경에 견딜 수 있도록 하는 것이 어려움
    - 인프라의 워크 플로우
        - Code(코드를 쓰기)
        - Review(검토한다)
        - Validate(검증)
        - Apply(적용)
    - Terraform 스테이트 시스템
        - Code에서 스테이트를 기술
        - Terraform측의 스테이트와 AWS측의 실제 스테이트가 다를 수 있다
    - Terraform스테이트 계승
        - VPC에서 Web Server, Web Server에서 ELB처럼 스테이트를 계승할 수 있는
        - 예)VPC에서 작성한 Subnet의 ID를 Web Server에서 지정
    - Terraform의 기능 확장
        - Power DNS
            - DNS 엔트리 작성
            - 커스텀 JWT(JSON Web Token)인증
            - Route 53처럼 쓸 수 있다
        - F5 configuration
            - VIP 작성
            - 자동으로 서버를 추가
            - ELB/ALB처럼 쓸 수 있다
        - MAAS provider
            - 베어 메탈 배포
            - 사용자 데이터
            - 배포하는 OS 정의
            - EC2 인스턴스와 같이 사용할 수 있다
    - 독자적인 도구를 만들
        - 어떤 때 독자적 도구를 만드는가?
            - 틈새 문제를 해결
            - 지금 있는 도구가 워크 플로에 맞지 않는다
            - 모든 상황에서 완전 통제
            - 툴 팬?
        - 무엇을 사용하여 독자적 도구를 만드는가?
            - 자신의 생태계 내의 언어
            - 자신의 조직에서 지원되는 언어
            - 당신이 학습한 언어?
- 아키텍처
    - Platform/도구/Mock Game Server/Test Clients는 AWS
    - 나머지는 DC
    - DC와 AWS는 DX에서 접속
    - 기능별로 VPC를 나누고 있다
    - 상호 접속이 필요한 각 기능은 VPC Peering로 접속
    - Shared services
        - DNS
        - NTP
        - Chef
        - AutoScaling Proxy
    - Platform
        - FrontEnd Apps
        - BackEnd Apps
        - Store Services
        - Matchmaking
        - Chat Services
        - Persistance Data Stores
    - Platform 2
        - Store DB
        - Matchmaking DB
        - Chat DB
    - Load testing harness
        - LT Harness를 대량 병렬로 프로비저닝
    - Load testing game servers
        - Game server를 몇대 나란히 프로비저닝
    - Load testing의 흐름
        - LT harness에서 Internet Gateway로
        - Internet을 지나 Riot DC로
        - DX를 지나 VPC로
    - 장기 운용에서의 과제
        - VPN Gateway의 상실
        - 인스턴스가 스택
        - 인스턴스의 증감
        - ELB의 스케일링
        - DX의 flapping
        - 볼륨 상실  
  
  