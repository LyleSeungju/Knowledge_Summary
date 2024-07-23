### Kubernetes 실습 및 개념 정리

## 어제 실습

### `kubectl --kubeconfig=dev-config` 명령어
- 이 명령어는 특정 kubeconfig 파일을 사용하여 Kubernetes 클러스터에 접근합니다.
- `microk8s kubectl` 명령어는 `kubectl --kubeconfig=/var/snap/microk8s/current/credentials/kubelet.config`와 동일합니다.

### 노드 정보 확인
```bash
kubectl describe node <node-name>
```

### 파드는 Kubernetes의 최소 단위
- `kubectl get pods -o wide`를 사용하면 파드가 어떤 애플리케이션이며, 어느 노드에 배포되었는지 확인할 수 있습니다.
- `kubectl get nodes -o wide`를 사용하면 각 노드의 정보를 확인할 수 있습니다.
  - 마스터 노드에는 배포된 애플리케이션이 없고, 워커 노드의 IP와 파드의 IP를 비교하여 배포된 위치를 알 수 있습니다.

### 명령어 설명
- `kubectl get svc -o wide`: 모든 서비스의 상세 정보를 출력합니다.
- `kubectl get daemonset -A`: 모든 네임스페이스의 DaemonSet 정보를 출력합니다.

### 리소스 == 오브젝트 == 종류
- Pod, Deployment, DaemonSet, CronJob 등 다양한 오브젝트가 있습니다.
- 각 오브젝트는 특정 용도에 맞게 사용됩니다.

### `snap` 명령어
- Snap은 애플리케이션 패키지 관리 시스템입니다.
- `/var/snap/microk8s/current` 디렉토리에서 MicroK8s의 환경 및 설정을 확인할 수 있습니다.
- `export` 명령어는 환경 변수를 설정하는 데 사용됩니다.

## Kubernetes 주요 오브젝트

### Control Plane (마스터 노드)
- Control Plane은 클러스터를 관리하고 명령을 실행하지만, 실제로 동작하지는 않습니다.
- API를 통해 각 서버에게 명령을 보냅니다.
- 주요 기능:
  - RESTful API
  - 사용자 인증 및 권한 부여
  - 리소스 유형 및 버전 관리
  - 스케줄링

### Controller
- 노드와 파드의 상태를 관리하고 감시합니다.
- 클러스터의 안정성과 신뢰성을 보장합니다.

### kube-scheduler
- 노드에 파드를 할당합니다.
- 새로 생성된 파드를 감지하고 적절한 노드에 배정합니다.
- 스케줄링 단계:
  - Filtering: 조건에 맞는 노드를 필터링
  - Scoring: 필터링된 노드를 점수화하여 최적의 노드를 선택

### etcd
- 클러스터의 모든 상태 정보를 저장하고 관리합니다.
- 높은 신뢰성을 제공합니다.

### Worker Node

#### kubelet
- 각 노드에서 실행되는 에이전트로, 파드가 동작하도록 관리합니다.
- 주요 기능:
  - 파드 관리
  - 컨테이너 실행
  - 리소스 모니터링
  - 상태 보고
  - 노드의 자동 복구

#### kube-proxy
- 파드 간의 네트워크 통신을 지원합니다.
- 동작 모드:
  - iptables mode
  - IPVS mode

#### Container runtime
- 파드 내에서 컨테이너를 실행합니다.

## Kubernetes Service
- 파드는 임시적이며, 언제든 종료되고 재시작될 수 있어 IP가 자주 바뀝니다.
- 고정 IP를 제공하는 서비스를 통해 파드에 안정적으로 연결할 수 있습니다.

### 서비스 종류
- ClusterIP: 클러스터 내부에서만 접근 가능
- NodePort: 외부에서 노드 IP의 특정 포트로 접근 가능 (30000~32767 사이 포트 사용)
- LoadBalancer: 클라우드 제공자가 지원하는 로드밸런서 사용

### Ingress
- HTTP 기반의 L7 로드밸런싱 기능을 제공합니다.

## Kubernetes Storage
- 컨테이너 내의 디스크는 임시적이며, 항상 초기화된 상태로 실행됩니다.

### Volume 종류
- hostPath: 호스트 노드의 파일시스템과 마운트
- emptyDir: 파드가 노드에 생성될 때 생성되며 파드가 삭제되면 함께 삭제됨
- ConfigMap: 설정값을 파드에 주입
- Secret: 암호와 같은 민감한 정보를 파드에 전달

### PersistentVolume (PV)
- 관리자가 스토리지를 미리 프로비저닝하여 제공
- 정적 스토리지

### StorageClass
- 동적 스토리지 프로비저닝을 지원

## Kubernetes 전체 개념
- 쿠버네티스는 클러스터 단위로 구성되며, 클러스터 내에는 마스터 노드와 워커 노드가 유기적으로 작동합니다.
- 각 파드 내에서 컨테이너가 독립적으로 실행됩니다.
- kubelet이 도커를 통해 파드를 관리하며, 파드는 컨테이너의 논리적 묶음입니다.
- 쿠버네티스 오브젝트는 리소스를 관리하기 위한 최소 단위입니다.

### Storage 관리
- hostPath와 emptyDir은 각 노드에서 관리됩니다.
- PV와 PVC는 외부 스토리지이지만 결국 노드에서 관리됩니다.
- EFS 같은 네트워크 디바이스는 예외입니다.
- 로컬 스토리지를 사용하는 경우 보안에 신경 써야 합니다.

## 실습

### 사전 준비 파일
- `deployment.yaml`
- `service.yaml`
- `service-nodeport.yaml`

### Deployment 생성
1. `deployment.yaml`
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     labels:
       app: nginx
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:1.14.2
           ports:
           - containerPort: 80
   ```
   ```bash
   kubectl apply -f deployment.yaml
   kubectl get deployment
   kubectl get pods -o wide
   ```
   - 배포된 파드의 IP를 확인하고 `curl <파드 IP>`로 접근합니다.

2. `service.yaml`
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service
   spec:
     selector:
       app: nginx
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```
   ```bash
   kubectl apply -f service.yaml
   kubectl get service
   kubectl get svc -o wide
   kubectl exec -it <nginx-pod-name> -- /bin/bash
   ```
   - `curl <서비스 IP>`로 접근한 후 `kubectl logs <nginx-pod-name>`로 로그를 확인합니다.

3. `service-nodeport.yaml`
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service-nodeport
   spec:
     type: NodePort
     selector:
       app: nginx
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
       nodePort: 30007
   ```
   ```bash
   kubectl apply -f service-nodeport.yaml
   kubectl get svc
   ifconfig | grep inet
   curl <노드 IP>:30007
   ```

### 포트 포워딩
```bash
kubectl port-forward <pod-name> <local-port>:<pod-port>
```

### MicroK8s 설치 및 제거
```bash
sudo snap remove microk8s
sudo snap install microk8s --classic
```
