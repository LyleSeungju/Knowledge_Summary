# Kubernetes Security

**개요:**
- Kubernetes는 Docker보다 보안에 더 많은 신경을 써야 합니다. 잘못된 권한 설정으로 인해 클러스터 전체가 해킹될 수 있으며, API 요청으로 노드를 제어할 수 있기 때문입니다.

### JWT를 공부하기 전 Cookie, Session, Token에 대한 이해
JWT(Json Web Token)을 이해하기 전에 Cookie, Session, Token의 차이점과 사용법을 알아봅니다:

- **쿠키(Cookie):** 클라이언트 측에 저장되는 작은 데이터 조각으로, 세션 관리, 사용자 추적 및 사용자 선호도 저장에 사용됩니다.
- **세션(Session):** 서버 측에서 사용자 데이터를 저장하여 사용자의 상호작용을 관리하고 추적합니다. 서버는 세션 데이터를 유지하고, 클라이언트는 세션 ID만 보유합니다.
- **토큰(Token):** 인증 정보를 포함하는 암호화된 문자열로, 서버는 토큰을 통해 사용자를 인증합니다. JWT는 이러한 토큰의 한 종류입니다.

### RBAC (Role-Based Access Control)
RBAC는 Kubernetes 리소스에 대한 접근 권한을 사용자나 그룹에게 부여하는 보안 메커니즘입니다.

- **명령어 예시:**
  - `kubectl config view --raw`
  - `kubectl get pods`
  - `kubectl apply`
  - `kubectl create`
  - `kubectl delete`

- **구성 요소:**
  - **Role** 및 **ClusterRole:** 네임스페이스와 클러스터 단위의 차이
    - **Rules:**
      - `apiGroups`: 리소스가 속한 API 그룹 및 버전
      - `resources`: 규칙을 적용할 리소스 목록
      - `verbs`: 리소스 접근 동사
  - **RoleBinding** 및 **ClusterRoleBinding:** 어디에 바인딩할 것인가의 차이
    - `subjects`: 권한을 부여할 사용자나 그룹
    - `roleRef`: 참조할 역할

- **ServiceAccount:** Kubernetes에서 애플리케이션의 인증 및 권한 관리를 담당하는 객체
  - `kubectl get serviceaccount`
  - 파드 내에서 `/var/run/secrets/kubernetes.io/serviceaccount` 경로에서 토큰을 볼 수 있습니다.

### Kubernetes 해킹
Kubernetes에서는 컨테이너를 넘어 클러스터 전체를 장악할 수 있습니다.

1. 간단한 nginx 파드 배포
2. `kubectl describe pod`로 파드 정보 확인
3. 파드에 접속하여 `/var/run` 디렉토리 탐색
4. `secrets` 디렉토리 내의 토큰 정보 확인
5. 토큰을 사용하여 API 요청으로 클러스터 정보를 조회

**팁:** `cat file.txt | pbcopy`를 사용하여 파일 내용을 클립보드에 복사할 수 있습니다.

### Kubernetes 자주 사용되는 용어

- **Taint:** 특정 노드에 파드가 스케줄되지 않도록 설정
  - **NoSchedule:** 해당 노드에 파드가 스케줄되지 않음
  - **PreferNoSchedule:** 가능하면 해당 노드에 스케줄되지 않음
  - **NoExecute:** 설정된 조건을 충족하지 않으면 파드를 종료

- **Toleration:** 파드가 특정 Taint를 무시하도록 설정

- **InitContainer:** 파드의 메인 컨테이너가 실행되기 전에 먼저 실행되는 컨테이너

- **Affinity / Anti-Affinity:** 파드의 스케줄링 규칙을 정의하여 특정 노드에 배치되거나 배치되지 않도록 제어

- **Custom Resource:** 사용자 정의 리소스를 생성하여 Kubernetes에서 사용할 수 있음

- **HPA (Horizontal Pod Autoscaler):** 파드의 수를 자동으로 확장
- **VPA (Vertical Pod Autoscaler):** 파드의 리소스를 자동으로 조정

### 클러스터 내에서 노드와 네임스페이스의 차이점
- **노드:** 물리적 리소스를 나누는 단위 (CPU, 메모리 등)
- **네임스페이스:** 논리적 오브젝트를 나누는 단위 (파드, 디플로이먼트 등)

### Kubeadm, Kubespray
- **Kubeadm:** Kubernetes 클러스터를 쉽게 설치하고 구성할 수 있도록 도와주는 도구
- **Kubespray:** Ansible을 사용하여 Kubernetes 클러스터를 설치하고 관리하는 도구

## 실습 1 

1.  사전 환경
    - 모든 노드에 RBAC 애드온 활성화되어 있는지 확인
        - `sudo microk8s enable rbac`
        - 오류나면 재설치
            
            `sudo snap remove microk8s --purge`  
            
            `sudo snap install microk8s --classic`
            
    - `mkdir practice` : 폴더 하나 만들어서 하는거 추천
2. Namespace 생성
    
    ```bash
    # ex-namespace 네임 스페이스 생성
    kubectl create namespace ex-namespace
    ```
    
3. ServiceAccount 생성
    
    ```yaml
    #serviceaccount.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: test-sa
      namespace: ex-namespace
      
    # sa 생성 
    k apply -f serviceaccount.yaml
    # 생성 확인
    k get serviceaccount -n ex-namespace
    k get sa -n ex-namespace
    ```
    
4. Secret 생성
    
    ```yaml
    #secret.yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: test-secret
      namespace: ex-namespace
      annotations:
        kubernetes.io/service-account.name: test-sa
    type: kubernetes.io/service-account-token
    
    #secret 생성
    k apply -f secret.yaml
    # 생성 확인
    k get secret -n ex-namespace
    ```
    
5. Role 생성
    
    ```yaml
    #role.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: ex-namespace
      name: pod-reader
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]
      
    # role 생성
    k apply -f role.yaml
    # 생성 확인
    k get role -n ex-namespace
    ```
    
6. RoleBinding 생성
    
    ```yaml
    #rolebinding.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: read-pods
      namespace: ex-namespace
    subjects:
    - kind: ServiceAccount
      name: test-sa
      namespace: ex-namespace
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io
      
    # rolebinding 생성
    k apply -f rolebinding.yaml
    # 생성된 목록 확인
    k get rolebinding -n ex-namespace
    # 생성된 rolebinding 상세 정보 조회
    k describe rolebinding read-pods -n ex-namespace
    ```
    
7. Pod 생성
    
    ```yaml
    #pod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pod
      namespace: ex-namespace
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sleep", "3600"]
        
    # pod 생성
    k apply -f pod.yaml
    # 생성 확인
    k get pods -n ex-namespace
     
    
    ```
    
8. Token값으로 API 호출하여 테스트
    
    ```yaml
    # 토큰값 가져오기 - secret에서 base64 디코딩해서 가져옴
    # 파드 생성 안하고도 가능한가 싶었는데 그럼 파드 리스트 암것도 없음
    k get secret test-secret -n ex-namespace -o jsonpath="{.data.token}" | base64 --decode
    
    Token=<복사한 토큰 값>
    
    #ex-namespace 기준 pods 리스트 받아오기 > 성공해야함 
    curl https://localhost:16443/api/v1/namespaces/ex-namespace/pods --header "Authorization: Bearer $Token" -k
    
    #kube-system 기준 pods 리스트 받아오기 > 권한 거절 당해야함
    curl https://localhost:16443/api/v1/namespaces/kube-system/pods --header "Authorization: Bearer $Token" -k
    
    #특정 네임 스페이스에 속한 sa를 생성하고, 얘를 통해 api를 요청해보는 실습
    #특정 네임 스페이스에서만 작동하는 역할을 지정하고, sa와 바인딩 시켜 sa는 특정 리소스에 대해 접근 할 수 있음.
    ```
