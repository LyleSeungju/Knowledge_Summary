## Service
#### IaaS
- AWS, GCP, Kakao Cloud
#### PaaS
- Heroku, Vercel, Netlify, 카페24
#### SaaS
- 구글 드라이브, 드롭박스, 네이버 MyBox

## Architecture
### 구성
#### OS  - operating system
- Windows, Mac, Linux
#### File System
- HDD, SSD etc.
- NFS(Network File System), NBD(Network Block Device)
#### VM - virtual machine
- OnDemand
- Reserved
- Spot Instance
#### LB - load balancer 
- Pre-Warming / Warm-Up
#### DB - data base
- DynamoDB
- Amazon RDS for MySQL, Elasticache
#### CDN - content delivery network
- resource caching
#### MSA - micro service architecture

### 종류
- 단일 아키텍처: Monolithic Architecture
- 마이크로서비스 아키텍처: Microservices Architecture (MSA)
- 계층형 아키텍처: Layered Architecture
	- 헥사고날 아키텍처: Hexagonal Architecture
- 이벤트 기반 아키텍처: Event-Driven Architecture
- 서버리스 아키텍처: Serverless Architecture
- 서비스 지향 아키텍처: Service-Oriented Architectur(SOA)
- 분산 아키텍처: Distributed Architecture
- 멀티티어 아키텍처: Multi-Tier Architecture
- 클라이언트-서버 아키텍처: Client-Server Architecture

### Multi-Tier Architecture
#### 1-Tier Architecture
1 server
- Front-End Server / Back-End Server / Database
#### 2-Tier Architecture
2server
- Front-End / Back-End
- Database
#### 3-Tier Architecture
3server
- Front-End
- Back-End
- Database
#### N-Tier Architecture
-> Final Architecture SaaS

## Cloud Tech Stack
![[Pasted image 20240714150438.png]]
#### 빌드 환경 구축 
 - CI / CD : Jenkins, GitLab CI/CD, CircleCI, Travis CI, TeamCity, Bamboo, Git Bucket
 - 빌드 도구: Maven, Gradle, Ant
 - 컨테이너: Docker, RKT, CRI-o
#### 배포 환경 구축
- CI / CD : Jenkins, GitLab CI/CD, CircleCI, Travis CI, TeamCity, Bamboo, Git Bucket
- 컨테이너 오케스트레이션 : Kubernetes, Docker Swarm, GCP GKE(google kubernetes service), AWS EKS(elastic kubernetes service), AWS ECS(elastic container service)
- 클라우드 서비스: AWS Elastic Beanstalk, Azure Devops, Google Cloud Build
#### 권한 관리
- IAM(Identity and Access Managemnet): AWS IAM, Azure AD, Google Cloud IAM
	- PAM 
- RBAC(Role-Based Access Control): Kubernetes RBAC, Keycloak, OpenLDAP
- ABAC(Attribute-Base Access Control): Axiomatics, NextLabs
#### 모니터링
- 모니터링: Prometheus, Grafana, Nagios, Datadog, New Relic
- 로그 분석 도구: ELK Stack (Elasticsearch, Logstash, Kibana), Splunk, Fluentd
	-  https://www.oss.kr/info_techtip/show/93661cc3-1f20-413e-b80c-c80164342b9b
### 자동화
- 인프라 자동화: Terraform, Ansible, Chef, Puppet
- CI / CD 파이프라인: Jenkins, GitLab CI/CD, TeamCity
#### 장애 대응
- 모니터링 및 경고: Prometheus, Grafana, PagerDuty, OpsGenie
- 장애 대응: Runbook Automation, Chaos Engineering 도구 (Gremlin)
- + 5Whys
#### 보안
- 클라우드 보안: AWS Security Hub, Azure Security Center, Google Cloud Security Command Center
- 네트워크 보안: VPN, VPC, WAF (Web Application Firewall)
- 데이터 암호화: KMS (Key Management Service), SSL/TLS
#### 서버 구축
- 클라우드 인프라: AWS EC2, Azure VM, Google Compute Engine
- 서버 자동화: Ansible, Chef, Puppe
- 컨테이너: Docker, Kubernetes
#### 인프라 운영
- 인프라 관리: Terraform, Ansible, Chef, Puppet
- 모니터링: Prometheus, Grafana, Datadog

#### 성능/비용 최적화
 - 비용 관리: AWS Cost Explorer, Azure Cost Management, Google Cloud Cost Management
 - 성능 최적화: Auto Scaling, Load Balancer, CloudFront(CDN)



## 프로젝트 진행 순서 및 정리

### 1. 프로젝트 초기 기획 및 설계
1. **요구사항 분석**
   - 클라이언트 및 이해 관계자와 협력하여 요구사항 수집
   - 사용자 스토리 및 기능 요구사항 정의

2. **시스템 설계**
   - 아키텍처 설계 (헥사고날 아키텍처, 마이크로서비스 아키텍처 등 선택)
   - 데이터베이스 설계 (RDBMS, NoSQL 선택)
   - 서비스 인터페이스 설계 (API 설계)

3. **기술 선택**
   - 서버 운영체제: Debian, Redhat, Windows
   - 데이터베이스: MySQL(MariaDB), PostgreSQL, Oracle Database, Microsoft SQL Server, MongoDB, Cassandra, Redis, DynamoDB
   - 프록시서버: Nginx, HAProxy, Apache HTTP Server
   - 로그시스템: ELK Stack, Splunk, Fluentd
   - 웹 프레임워크:
     - Java: Spring, Spring Boot
     - JavaScript: Node.js, Express.js, React, Angular, Vue.js
     - Python: Django, Flask, FastAPI
     - Ruby: Ruby on Rails
     - PHP: Laravel, Symfony
4. **프로토타입 개발**
   - 주요 기능의 프로토타입 개발
   - 초기 테스트 및 피드백 수집

### 2. 개발 단계
1. **개발 환경 구축**
   - 개발 서버 구성 (AWS EC2, Azure VM, Google Compute Engine, Docker)
   - 소스 코드 관리 (Git)
   - 빌드 및 배포 도구 설정 (Jenkins, GitLab CI/CD, Docker, Kubernetes)
   - 로깅 시스템 설정 (ELK Stack, Splunk, Fluentd)
2. **코딩 및 테스트**
   - 기능별 모듈 개발
   - 단위 테스트 및 통합 테스트 작성
   - 코드 리뷰 및 지속적 통합 (CI/CD 파이프라인)
3. **Docker 환경 구성**
   - Dockerfile 작성 및 최적화
   - Docker Compose 설정
   - Docker Hub에 이미지 저장 및 관리

### 3. 배포 및 운영
1. **배포 및 릴리즈 관리**
   - 자동화된 배포 파이프라인 구축 (Jenkins, GitLab CI/CD)
   - 스테이징 환경 및 프로덕션 환경으로의 배포
   - 릴리즈 노트 작성 및 배포 일정 관리
2. **모니터링 및 유지보수**
   - 모니터링 도구 설정 (Prometheus, Grafana)
   - 실시간 로그 모니터링 (ELK Stack, Splunk, Fluentd)
   - 장애 대응 프로세스 수립 및 대응 (Prometheus, Grafana)
3. **백업 및 복구 전략**
   - 정기적인 데이터 백업 계획 수립
   - 장애 발생 시 복구 절차 수립

### 4. 자동화 및 최적화
1. **CI/CD 파이프라인**
   - 코드 변경 시 자동 빌드 및 테스트
   - 자동 배포 및 롤백 기능 구현
2. **인프라 관리 (IaC)**
   - 인프라 자동화 도구 사용 (Terraform, Ansible)
   - 버전 관리 및 코드 리뷰 적용
3. **테스트 자동화**
   - 자동화된 테스트 스크립트 작성 및 유지보수
   - 성능 테스트 및 부하 테스트 자동화

### 5. 새로운 기능에 대한 리서치 및 테스트
1. **기능 연구 및 파일럿 프로젝트**
   - 새로운 기술 및 기능에 대한 리서치
   - 파일럿 프로젝트를 통한 초기 테스트 (AWS Lambda, Azure Functions, Google Cloud Functions, Serverless Framework)

2. **피드백 수집 및 개선**
   - 사용자 및 팀의 피드백 수집
   - 기능 개선 및 최적화
### 6. 팀 프로젝트 주제 검토 및 최종 점검
1. **팀 프로젝트 주제 검토**
   - 프로젝트 진행 상황 점검
   - 각 단계별 완료된 작업 검토 및 피드백

2. **최종 점검 및 릴리즈 준비**
   - 최종 테스트 및 품질 확인
   - 배포 준비 및 최종 릴리즈
