#### EC2 Basic
- OS
- CPU
- RAM
- Storage
	- EBS, EFS
	- EC2 instance store
- Network card
- Firewall rules: 보안그룹
- Bootstrap script : 머신이 작동할떄 명령을 시작하는 것
	- 처음 인스턴스가 생성될때 부팅 작업을 자동화하기 때문에 "부트스트래핑"
	- ex. 설치, 업데이트 등등

#### AWS EC2 Instance Create
- 맨 밑에 사용자 데이터(고급)에서 맨 부팅되면 실행시킬 명령어들을 지정 할 수 있다.

**Q. ec2 인스턴스 생성 때 사용 할 수 있는 Boot strap 기능으로 도커를 설치하는것과 Ansible로 도커를 설치하는 것과 어느것을 쓰는편?**

```
우분투 기준
#! /bin/bash
sudo apt-get update -y
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings -y
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.as
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
```


```
아마존 리눅스 기준
#!/bin/bash
yum update -y
yum install -y docker
systemctl start docker
systemctl enable docker
usermod -aG docker ec2-user
```

#### AWS Security Group
- 포트 제한
- IP 범위 제한 - IPv4 / IPv6
- 인바운드
- 아웃 바운드

알면 좋은 것들 
- 타임아웃(time out): 보안그룹에서 포트 안열린 경우
- 연결거부(connection refused): 포트는 연결되어 있는데 애플리케이션이 오류가 나거나 실행이 되지 않은 것
- 기본적으로 모든 인바운드 트래픽은 차단되고 아웃 바운트 트래픽은 승인
	- 이것을 통해 인스턴스와 인스턴스끼리의 통신을 할때 보안그룹에서 서로 동일한 포트를 지정하는게 필요한 것이 아니라 들어오는 인바운드 규칙만 신경 쓰면 됨!!!

기본적인 포트 
- 22 - SSH
- 21 - FTP
- 22 - SFTP
- 80 - HTTP
- 443 - HTTPS
- 3389 - RDP : 윈도우 인스턴스 접속용

#### AWS EC2 Instance security
- 인스턴스에 IAM 역할을 정해줄 수 있는데 이것을 통해 IAM 사용자를 입력해줄 필요 없이 해당 인스턴스에만 특정 권한을 부여해줄 수 있다.

#### AWS Instance 플랜
- On demand 
	- 필요할 때 즉시 EC2 인스턴스를 시작하고, 시간 단위로 비용을 지불하는 방식입니다. 인스턴스를 언제든지 시작하고 종료할 수 있으며, 장기적인 약정이나 선불 결제 없이 사용한 만큼만 지불합니다.
- Reserved - 내가 1년/3년동안 쓸 인스턴스를 미리 구매해두는건데 비싼편
	- 특정 유형의 인스턴스를 1년 또는 3년 단위로 예약하여 장기 사용을 약정하고, 이에 따른 할인 혜택을 받는 방식입니다. 인스턴스 유형, 운영 체제, 리전 등을 예약 시점에 미리 선택하게 됩니다.
- Savings Plan - 내가 1년/3년동안 시간당 쓸 비용을 미리 예상해두고 할인 받는 개념(정확할수록 좋음)
	- 일정 기간 동안 일정 금액의 사용량을 약정함으로써 할인 혜택을 받는 구매 옵션입니다. Compute Savings Plans와 EC2 Instance Savings Plans의 두 가지가 있으며, 전자는 EC2뿐 아니라 AWS Fargate, Lambda 등에도 적용될 수 있습니다.
- Spot Instance - 언제든 뺏길수 있지만 경매방식이기때문에 많이 쌈
	- AWS에서 유휴 상태인 컴퓨팅 용량을 경매 방식으로 제공하는 인스턴스입니다. On-Demand보다 훨씬 저렴하지만, AWS가 해당 용량이 필요할 경우 언제든지 인스턴스를 중지시킬 수 있습니다.
- Dedicated Hosts
	- 물리적 서버 전체를 전용으로 사용하는 인스턴스 옵션입니다. 특정 하드웨어에 대한 물리적 서버를 전용으로 사용할 수 있어 라이선스 요구 사항을 충족시키거나, 특정 보안 및 규제 요구 사항을 충족할 수 있습니다. 
- Capacity Reservations
	- 특정 리전 내의 특정 인스턴스 유형에 대해 예약된 용량을 확보하는 방식입니다. 필요할 때 사용할 수 있도록 인스턴스 용량을 미리 예약해 두는 것으로, EC2 인스턴스를 보장된 용량으로 사용할 수 있습니다.

#### AWS Spot Instance
2가지 옵션
- max spot price: 해당 가격을 초과하면 인스턴스를 중지하고 다시 해당하면 실행시키게끔
- spot block, 스팟 인스턴스를 회수되지 않게

스팟 인스턴스 종료 방법
1. 스팟 요청
	- 원하는 인스턴스 수
	- 최대 가격
	- 시작 사양
	- 기한
	- 요청 유형 2가지
		- 일회성 요청: 스팟 요청이 완료되는 즉시 인스턴스가 시작 > 스팟 요청이 사라짐
		- 영구 인스턴스 요청 > 스팟 요청이 유효한 동안 인스턴스가 멈춰도 다시 스팟 요청에 의해 실행됨
2. 인스턴스를 종료하려면?
	- 스팟 요청 취소 > 인스턴스 종료

#### Spot Fleets
AWS에서 다양한 인스턴스 유형과 구매 옵션(스팟 인스턴스, 온디맨드 인스턴스)을 조합하여 목표 용량을 충족시
키기 위해 사용되는 서비스

특징
- 다양한 인스턴스 유형 및 풀 선택
	- 사용자는 여러 인스턴스 유형과 가용 영역을 지정하여 플릿이 목표 용량을 달성할 수 있도록 함
- 자동 최적화
	- 플릿은 가장 적합한 런치 풀에서 인스턴스를 선택
- 예산 및 용량 관리
	- 설정한 비용이나 용량에 도달하면 플릿은 인스턴스를 요청하지 않거나 인스턴스를 중지시킴

할당 전략
1. Lower Price (최저 가격): 플릿은 가장 저렴한 인스턴스 풀부터 인스턴스를 시작
2. Diversified(다양화): 플릿은 여러 인스턴스 풀에 분산시켜 할당
	- 특정 풀에 의존하지 않아 중단 위험이 분산
3. Capacity Optimized(용량 최적화): 플릿은 용량이 가장 많은 인스턴스 풀에서 인스턴스를 선택 
	- 중단될 확률이 낮아짐
4. Price Capacity Optimized(가격 및 용량 최적화): 플릿이 가격과 용량을 고려하여 최적의 인스턴스 풀을 선택
	- 가장 추천하는 옵션


#### 퀴즈 
1. 다음 중, 할인 폭이 가장 크나, 데이터베이스 혹은 중요 업무에는 적합하지 않은 EC2 구매 옵션은 무엇인가요?
	- 전환 가능 예약 인스턴스 > reserved : 내가 미리 사놓고 새로운 요구사항이 생기면 다른 유형으로도 바꿀 수 있다.
		- 전환 가능 예약 인스턴스는 유연한 인스턴스를 가진 장기적인 워크로드에 적합합니다. 하지만 할인율이 가장 높지는 않습니다.
	- 전용 호스트(Dedicated Hosts) - 가장 비쌈
	- 스팟 인스턴스 
		- 근데 신뢰도가 가장 낮음
2. EC2 인스턴스 내/외의 트래픽을 제어하기 위해서는 무엇을 사용해야 하나요?
	- 네트워크 액세스 제어 리스트 NACL
		- VPC 서브넷에서 동작하고 서브넷 내의 모든 인스턴스에 대한 트래픽을 제어 
	- 보안 그룹
	- IAM 정책
3. EC2 예약 인스턴스를 예약할 수 있는 기간은 얼마인가요?
	- 1년 혹은 3년
	- 2년 혹은 4년
	- 6개월 혹은 1년
	- 1년에서 3년사이의 기간
4. EC2 인스턴스에 고성능 컴퓨팅(HPC) 애플리케이션을 배포하려 합니다. 다음 중 어떤 EC2 인스턴스 유형을 선택해야 할까요?
	- 스토리지 최적화
	- 컴퓨팅 최적화
		- 컴퓨팅 최적화 EC2 인스턴스는 고성능 프로세서(예: 배치 처리, 미디어 트랜스코딩, 고성능 컴퓨팅, 과학적 모델링 및 머신 러닝, 전용 게이밍 서버 등)가 필요한 집중 컴퓨팅 워크로드에 적합합니다.
	- 메모리 최적화
	- 범용
5. 1년 간 지속적으로 서버를 운영할 계획인 애플리케이션의 경우에는 다음 중 어떤 EC2 구매 옵션을 사용해야 할까요?
	- 예약 인스턴스
	- 스팟 인스턴스
	- 온디맨드 인스턴스 
6. 일련의 EC2 인스턴스에 호스팅 될 애플리케이션을 실행하려 합니다. 이 애플리케이션에는 소프트웨어 설치가 필요하며, 최초 실행 중에 일부 OS 패키지를 업데이트해야 합니다. EC2 인스턴스를 실행하려는 경우, 이를 위한 최적의 방식은 무엇일까요?
	- SSH를 통해 각 EC2 인스턴스에 연결 한 후, 필수 소프트웨어를 설치하고 OS패키지를 수동으로 업데이트
	- 필수 소프트웨어의 설치 및 OS업데이트를 수행하는 bash 스크립트를 작성 후,aws 지원 센터에 연락해서 지원해주기
	- 필수 소프트웨어의 설치 및 OS업데이트를 수행하는 bash 스크립트를 작성 후, 이 스크립트를 EC2 사용자 데이터에서 사용
		- EC2 사용자 데이터는 bash 스크립트를 사용해 EC2 인스턴스를 부트스트랩 할 경우에 사용됩니다. 이 스크립트에는 소프트웨어/패키지 설치, 인터넷에서 파일 다운로드 등, 여러분이 원하는 작업을 수행하기 위한 명령어를 포함시킬 수 있습니다.
7. 인메모리 데이터베이스를 사용하는 중요 애플리케이션을 위해서는 다음 중 어떤 EC2 인스턴스 유형을 선택해야 할까요?
	- 컴퓨팅 최적화
	- 스토리지 최적화
	- 메모리 최적화
	- 범용
8. 온프레미스에 호스팅된 OLTP 데이터베이스를 갖춘 전자 상거래 애플리케이션이 있습니다. 이 애플리케이션은 인기가 좋아, 데이터베이스가 초당 수천 개의 요청을 지니게 됩니다. 여러분은 데이터베이스를 EC2 인스턴스로 이전하려 합니다. 이렇게 높은 빈도를 보이는 OLTP 데이터베이스를 처리하기 위해서는 어떤 EC2 인스턴스 유형을 선택해야 할까요?
	- 컴퓨팅
	- 스토리지
		- 스토리지 최적화 EC2 인스턴스는 로컬 스토리지의 대규모 데이터 세트에 대해 높은 수준의, 그리고 순차적인 읽기/쓰기 액세스 권한이 필요한 워크로드에 적합합니다.
	- 메모리
	- 범용
9. (참/거짓) 보안 그룹은 오직 하나의 EC2 인스턴스에만 연결될 수 있습니다.
	- No
10. 온프레미스 애플리케이션을 AWS로 이전하려 합니다. 여러분의 기업에는 애플리케이션을 전용 서버에서 실행해야 한다는 엄격한 규정이 있습니다. 또한 비용 절감을 위해 전용 서버 바운드 소프트웨어 라이선스를 사용해야 합니다. 이 경우, 다음 중 어떤 EC2 구매 옵션이 적합할까요?
	- 전환 가능 예약 인스턴스 컨버터블 예약 인스턴스
	- 전용 호스트
		- 전용 호스트는 높은 수준의 규정 준수가 필요한 기업, 혹은 복잡한 라이선스 모델을 가진 소프트웨어에 적합합니다. 이는 가장 비싼 EC2 구매 옵션입니다.
	- 스팟 인스턴스
11. 데이터베이스 기술을 EC2 인스턴스에 배포하려 합니다. 공급 업체 라이선스는 물리적 코어 및 기반 네크워크 소켓 가시성를 기반으로 비용을 책정합니다. 이 경우, 어떤 EC2 구매 옵션을 사용해야 가시성을 확보할 수 있을까요?
	- 스팟 인스턴스
	- 온디맨드
	- 전용 호스트
	- 예약 인스턴스
12. 스팟 플릿은 스팟 인스턴스의 집합이며, 선택적으로 ___________ 이다.
	- 예약 인스턴스
	- 온디맨드 인스턴스 
		- 스팟 플릿은 스팟 인스턴스의 집합이며, 선택적으로 온디맨드 인스턴스입니다. 스팟 플릿은 가장 낮은 가격으로 스팟 인스턴스를 자동으로 요청할 수 있게 해줍니다.
	- 전용 호스트
	- 전용 인스턴스
