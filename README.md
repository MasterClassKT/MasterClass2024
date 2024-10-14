# [고수의 클래스] Azure온보딩 실습: KT사내 서비스 Azure Migration


<br/>

본 교육 과정은 컨테이너 기반 KT사내서비스를 Azure Cloud 환경에 온보딩하고 서비스 동작의 이상상태 유무 체크 그리고, Azure에서 제공하는 PaaS상품을 기존 오픈소스 기반의 서비스를 대체하여 활용까지 수행한다.     

문의 :  정치훈 ( ch.jung@kt.com / giglepeople@gmail.com )

<br/>


1. Session 1 :     ( [가이드 문서보기](./session1.md) ) 

     - Github & Docker 환경 구성
          - Github 설치
          - Github repository 나만의 branch 생성
          - Docker image push to ACR (로컬PC)
     - Azure 리소스 생성
          - ACR 생성
          - Eventhub 생성 (실습)
          - Redis 생성
          - PostgreSQL DB 생성
          - AKS 생성
     
     <br/>

2. Session 2 :     ( [가이드 문서보기](./session2.md) )

     - Eventhub Events만들기(실습)     
     - CI 파이프라인 만들기 
          - Github workflow 탐구 (Step by Step)
          - 나만의 워크플로우 만들기(실습)
          - 실습 과제
          - AKS - 나만의 네임스페이스 만들기

3. Session 3 :     ( [가이드 문서보기](./session3.md) )
   
   - Github 소스 수정 
          - Paas서비스와 연동을 위한 config 수정
          - 모니터링 구성 
               - WhaTap
               - Azure Insights 
          - 보안 취약점 조치
               - Springboot, jdk 버전 올리기
               - Trivy를 통한 Container 취약점 조치
          - 실습 과제

4. Session 4 :     ( [가이드 문서보기](./session4.md) )

   - AKS에 컨테이너 배포 테스트
          - yaml 만들기
          - 배포해보기(실습) 
               - PaaS서비스와 연동 체크
   
