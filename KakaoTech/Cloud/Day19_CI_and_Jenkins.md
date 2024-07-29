### EKS 구성 요소 및 실습

#### EKS 구성 요소
- **Control Plane**: AWS가 관리하고 보장하며, 사용자는 직접 볼 수 없음.
- **Worker Nodes**:
  - **CAS (Cluster Autoscaler)**: 오토스케일링 지원되지만 느림.
  - **Karpeneter**: 다양한 인스턴스 타입 지원.

#### EKS Add-ons
- **Load Balancer**
- **EBS CSI Controller**: 직접 설정하기 번거로움.
- **Pod 연결 시 Add-on 필수**

### EKS 실습
#### 0. Terraform으로 사전 환경 세팅 및 배포
1. **디렉토리 및 초기화**:
   ```sh
   mkdir practice
   cd practice
   terraform init
   terraform apply
   ```
2. **IAM 사용자 생성**:
   - IAM 사용자 생성 후 `AdministratorAccess` 정책 연결.
   - 생성된 액세스 키와 비밀 액세스 키를 `aws configure` 명령어로 설정.
   - Terraform 파일의 `key_name` 항목 수정 필요.

#### 1. EKS 클러스터 생성
1. **EKS 검색 및 클러스터 추가**:
   - 클러스터 이름, IAM 역할 생성, 최신 Kubernetes 버전 선택.
   - VPC 및 서브넷 선택 (Terraform으로 생성된 VPC 및 서브넷).
   - 보안 그룹 선택 (Terraform으로 생성된 보안 그룹).
2. **노드 그룹 생성**:
   - 이름 설정 후 권한 추가 (AmazonEKSWorkerNodePolicy, AmazonEC2ContainerRegistryReadOnly, AmazonEKS_CNI_Policy).
   - 노드 그룹 컴퓨팅 구성 (Amazon Linux, On-demand, t3 인스턴스 유형).

#### EKS 사용 설명서를 따라 설정 완료

### CI
#### CI의 중요성
- **빠른 피드백**: 빌드 및 테스트 자동화.
- **버그 감소**
- **개발 속도 증가**: 지속적 통합, 자동화된 테스트.
- **자동화**

#### CI 워크플로우
1. 코드 커밋
2. 자동화된 빌드
3. 자동화된 테스트
4. 통합 및 배포
5. 피드백 제공

#### 유명한 CI 도구
- **Jenkins**: 무료, CI/CD 지원.

### Jenkins 실습
0. **사전 세팅**: 위의 EKS 환경 설정.
1. **AWS 인스턴스 접속 및 Jenkins 설치**:
   ```sh
   # APT 키 추가
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null

   # 소스 목록 추가
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null

   # 패키지 목록 업데이트
   sudo apt update

   # Jenkins, OpenJDK, Docker 설치
   sudo apt install jenkins openjdk-11-jdk docker.io -y

   # Docker 소켓 소유권 변경
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   ```
2. **Jenkins 활성화**:
   ```sh
   sudo systemctl enable jenkins --now
   ```
3. **Jenkins 접속**:
   - 인스턴스 Public IP 주소 확인 후 브라우저에서 `http://<public-ip>:8080` 접속.
   - 초기 비밀번호 확인:
     ```sh
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
4. **어드민 사용자 생성**: 스킵 가능, 기본 정보 입력 후 진행.
5. **Jenkins 플러그인 설치**:
   - GitHub Integration, Docker Pipeline, Amazon ECR, AWS Credentials 설치.
6. **GitHub 레포지토리 연결 - 웹훅 설정**:
   - GitHub 레포지토리 설정에서 Webhooks 추가.
   - Payload URL: `http://<public-ip>:8080/github-webhook/`.
7. **GitHub 토큰 발급**:
   - GitHub에서 Personal Access Token 발급 (repo, admin:repo_hook 권한 선택).
8. **Jenkins에 GitHub 서버 설정**:
   - 글로벌 Credential 추가 후 연결 테스트.

### Jenkins에서 Credential 및 파이프라인 설정
1. **Credential 생성**:
   - AWS Credentials 생성 (ecr_credential_id, 액세스 키, 시크릿 액세스 키 입력).
2. **Pipeline 설정**:
   - Jenkins에서 새로운 아이템 생성 (Pipeline).
   - GitHub 프로젝트 URL 입력.
   - Build Trigger 설정 (GitHub webhook).
   - Pipeline 스크립트 입력 (ecr id 수정).
3. **GitHub 레포지토리에서 커밋 발생**:
   - 예: README.md 파일 수정 후 커밋.
   - Jenkins에서 빌드 시작.
4. **ECR에서 이미지 확인**: 정상적으로 생성된 이미지 확인.

### 앱 배포 및 확인 (맥 호스트)
1. **AWS CLI 및 kubectl 설치**:
   ```sh
   aws configure
   curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/arm64/kubectl"
   chmod +x ./kubectl
   sudo mv ./kubectl /usr/local/bin/kubectl
   aws eks update-kubeconfig --region ap-northeast-2 --name your-cluster
   kubectl apply -f deployment.yaml
   ```
2. **클러스터 내용 확인**:
   ```sh
   kubectl get pod
   ```

위 단계를 통해 EKS 설정 및 Jenkins를 이용한 CI/CD 파이프라인을 구성하고, 애플리케이션을 배포하는 과정을 진행할 수 있습니다.
