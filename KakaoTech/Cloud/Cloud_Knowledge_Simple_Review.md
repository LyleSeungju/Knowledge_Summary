
## 클라우드 개론

#### 가상화
1. 호스트 가상화 - os가 필요함 ex. VM
2. 하이퍼바이저 가상화 - Xen
	- Bare-Metal
	- Hosted Hypervisor
- **호스트 가상화와 하이퍼 바이저 가상화의 차이점?** 호스트 OS 위에 게스트 OS를 올릴 것이냐  vs. 하이퍼 바이저에 OS를 올릴것이냐
	- 호스트 가상화는 OS위에 새로운 OS가 올라가기때문에 OS가 2중으로 되어 불필요해짐
3. 컨테이너 가상화: OS위에 컨테이너 관리 소프트웨어로 별도의 OS 필요 없이 독립된 공간 제공

AWS 가상화: PV(paravirtualization) > HVM(Hardware Virtual Machine) > Nitro System

종류
- 서버 가상화: 하나의 물리적 서버 -> 여러 가상 서버
- 데스크탑 가상화: 데스크탑 환경 가상화
- 애플리케이션 가상화: 애플리케이션을 가상 환경에서 실행
- 네트워크 가상화 : 물리적 네트워크를 여러 가상 네트워크로
- 스토리지 가상화: 물리적 스토리지를 추상화하여 논리적 스토리지 풀

VM vs Bare Metal
- VM : 가상화 방식, 일부만 가상화하여 사용
- Bare Metal : 통째로 빌려주기 서버

IaaS - AWS, GCP, Kakao Cloud
PaaS - Vercel, Heroku
SaaS - 대부분의 클라우드 서비스

#### 클라우드 아키텍처를 위한 배경지식들
- OS
- 네트워크
- 파일시스템
- VM 
- LB
- 데이터베이스
- CDN
- MSA

#### 클라우드 아키텍처 구조
- 1티어 구조 
- 2티어 구조
- 3티어 구조
- N티어 구조

## 네트워크 기초

#### OSI 7 Layer
- 데이터 링크 계층
	- 이더넷
	- 브릿지
	- 스위치
- 네트워크 계층
	- IP 
	- CDIR
	- 서브넷
	- 라우터
	- NAT
- 전송 계층
	- tcp
	- udp
- 응용 계층
	- http


## 리눅스 기초
- 쉘
	- 명령어 해석
	- 스크립트 실행
	- 환경 관리
- 커널
	- 프로세스 관리
	- 메모리 관리
	- 파일 시스템 관리
	- 장치 관리
	- 보안 및 접근 제어
- 하드웨어
	- CPU
	- 메모리
	- 저장장치

#### OS
- Ubuntu
- CentOS
- Arch Linux
#### 권한
- 사용자 / 그룹
- Read, Write, eXecute
#### 디렉토리
- Root: 시스템 모든 자원
- /bin : 기본적인 명령어 - binary
- /lib : 공유 라이브러리 - Library
- /usr : 사용자 명령어, 응용 프로그램, 라이브러리 - Unix System Resource & User
- /dev: 장치파일 - Device
- /etc: 시스템 설정 파일 -  Et cetrera
- /opt : 설치된 애플리케이션 - Optional
- /proc: 커널과 프로세스에 대한 가상 파일 시스템 - Process
- /home: 사용자 별 개인 디렉토리 - home
- /tml: 임시파일 저장하는 디렉토리 - temporary
## 리눅스 심화
- cgroup - Control Group 
	- 리소스 제한
	- 리소스 우선 순위
	- 리소스 계정
	- 리소스 격리
#### Namespace
- 리소스 격리
- 컨테이너 기술에 사용
- 종류 
	- Mount Namspace (mnt)
	- Process ID Namespace(pid)
	- Network Namespace(net)
	- User Namespace(user)
	- IPC Namespace(ipc) - Inter process comunication
	- UTS Namespace(uts) - Unix time-sharing

- chroot
- unix domain socket(uds)
	- 로컬 통신
	- 파일 시스템을 이용한 주소 지정
	- 데이터 전송 속도
	- 보안
- OverlayFs (Overlay Filesystem)
	- 파일 시스템 계층화
	- 스냅샷 및 버젼 관리
	- 컨테이너
- 파일 시스템
	- Lowerdir - 읽기
	- Upperdir - 쓰기
	- Workdir - 변경사항 추적
	- Merged - lower + upper > 현재 상태의 파일 시스템 
	- 작동 방식
		1. 새로운 파일 생성 - upperdir 생성 
		   - uppder에만 파일 생성
		2. 기존 파일 읽기 - upperdir 있으면 upperdir 에서 읽고, 없으면 lowerdir 에서 읽음
		   - 파일이 생성될 때 upper만 생성되는데 lowerdir에 언제 파일이 들어가게 되는지?
		3. 기존 파일 수정 - upperdir 없으면 lowerdir에서 uppderdir로 복사 후 수정
		4. 파일 삭제 - upperdir 있으면 삭제되고, lowerdir에 있는경우 uppderdir 삭제 마커 생성

- **리눅스 파일시스템에서 새로운 파일 시스템이 생성되면 upperdir에 생성된다하는데, 정확히는 맨 처음 파일이 그 디렉토리에 위치하게 된다면 Lowerdir에 기록되고 그 이후에 변경사항은 모두 uppderdir에 적히는 건가요?**
## 도커 개론
- 환경에서의 이점
- 자원 관리
- 독립된 환경에서 작업
## 도커 기초

#### Dockerfile 특징
- 환경 일관성
- 이식성
- 자동화
- 반복 가능성
- 확장성

레이어 캐싱
- 필요한 것만 설치하라 개발 환경이 아니다
	- 불필요한 패키지 제외 - ex. npm prune, depcheck
	- 빌드 도구 제거
	- 정리 작업 수행
- 적절한 기본 이미지를 사용하라
	- 경량 베이스 이미지 사용
	- 필요한 도구가 포함된 이미지 사용
- 다단계 빌드를 사용하라 -> Multi-Staged Build
	- 빌드 도구와 종속성 설치는 빌드 스테이지에서 수행하고 최종은 빌드된 결과물만 포함
- 가능하면 명령어를 함께 써라
	- RUN 명령어 결합: 여러 명령어를 && 사용
	- 중간 파일 제거

Q. 실제 현업에서도 이러한(최적화) 작업을 필수로 고려하는지?


Docker Compose vs Kubernetes 
- 규모& 복잡성
	- Docker Compose : 단일 호스트에서 다중 컨테이너 애플리케이션 관리
	- Kubernetes: 분산된 클러스터 환경에서 대규모 애플리케이션을 관리
- 사용 사례
	- Docker compose : 로컬 개발 환경 및 간단한 테스트 환경 
	- Kubernetes : 프로덕션 환경에서의 대규모 배포, 자동화된 관리 필요할때
Q. 현업에서도 Docker Compose와 Kubernetes의 사용이 어떻게 되는지 궁금 / 쿠버네티스를 사용하는 곳과 아닌곳에서는 어떻게 Docker 를 쓰이는지? 

## 도커 보안 및 구조 

도커 
Privileged Container vs. Unprivilege Container
- Privileged Contianer: 호스트의 권한을 가지는 컨테이너 > 어디에 사용?
- Unprivileged Container: 대부분의 애플리케이션, 최소한의 권한

Security Layer
- AppArmor: 리눅스 보안 모듈, 프로세스 능력을 제한 
- SELinux  :보안 컨텍스트를 통해 권한 관리 
- Seccomp : 시스템 콜 필터링을 통해 컨테이너가 수행할 수 있는 시스템 콜 제한
- User Namespace: 컨테이너 내 사용자와 호스트 사용자 간 매핑
	- 사용자 ID 매핑(UID mapping): 사용자와 호스트 id를 분리
	- 그룹 ID 매핑(GID mapping): 그룹단위로 호스트 시스템의 그룹과 분리
	- 네임스페이스 격리
	- 보안 강화
Rootless mode: 컨테이너를 비루트 사용자로 실행하는 방식

## AWS


Q. ALB는 애플리케이션 접속을 분산 시켜주는 것이라는 것을 아는데 NLB는 어느때 써요? 
- VoIP & 게임 서버
- 실시간?
Q. ALB와 NLB를 함께 쓸때도 있나요?
- 외부 트래픽은 ALB,내부 데이터 베이스 트래픽은 NLB
Q. GLB도 쓸때가 있나요?

#### ABAC(속성 기반)와 기존 RBAC(역할 기반) 모델 비교

관리의 용이성 
- RBAC: 역할 기반 > 역할 수가 많으면 관리가 어렵
- ABAC: 속성 기반으로 관리 > 그냥 관리가 복잡
유연성
- R: 덜 유연
- A: 매우 유연
정책 
- R: 역할에 대해 미리 정의하고 접근을 제어
- A: 사용자, 리소스, 환경 등을 속성을 조합하여 세밀한 정책 정의
적용 예시
- R: 조직의 역할이 명확하고, 고정된 환경
- A: 다양한 조건과 상황에 따라 접근 제어가 필요한 환경


## Kubernetes

- 쿠버네티스는 기본적으로 컨테이너 런타임 위에 설치되고 쿠버네티스 컨테이너 안에서는 pod 단위로 새로운 컨테이너들을 띄운다.
bin packing - 기본적인 알고리즘 문제 -> 각기 다른 여러개의 아이템을 가능한 적은 수의 빈을 사용하여 모든 아이템을 빈에 배치하게끔 하는 것

Control Plane - 클러스터의 모든 구성 요소와 상호작용(API CRUD)
- What is 'etcd'
	- 모든 클러스터의 상태 저장
	- key-value 형태로 저장 -> 모든 클러스터 노드가 동일한 상태정보를 가질 수 있도록 보장
	- 높은 가용성 : 다른 노드가 실패해도 해당 데이터를 다 들고 있음
	- 변경 감지: 데이터 변경사항으 감지하고, 이를 기반으로 클러스터의 상태를 업데이트
	- [etcd에 좀 더 깊숙히 공부해보자](https://tech.kakao.com/posts/484)
- kube-scheduler
	- 
