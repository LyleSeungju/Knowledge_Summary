


# Kubernetes 실습

## 실습 1: 클러스터 구성
- 구성 환경: UTM Linux VM 환경 3개
  - EC2 인스턴스 환경(t2.micro)은 테스트용으로 성능이 부족함
  - 1개는 Master node
  - 2개는 Worker node

- permission denied가 발생하면 `sudo` 사용 또는 `sudo su -`로 루트 계정에서 진행

### 1. MicroK8s 설치
각 노드에 다음 명령어로 MicroK8s를 설치합니다:
```bash
sudo snap install microk8s --classic --channel=1.30
```
설치가 정상적으로 완료되었는지 확인합니다:
```bash
microk8s kubectl get pods -A
```

### 2. Master node에 Worker node 연결
- Master node에서 다음 명령어를 실행합니다:
  ```bash
  microk8s add-node
  ```
  출력된 `microk8s join 192.168~` 형태의 명령어를 복사합니다.

- Worker node에서 다음 명령어를 실행하여 Master node에 연결합니다:
  ```bash
  microk8s join <master-ip>
  ```

- 모든 노드를 연결한 후 다음 명령어로 연결된 노드 정보를 확인합니다:
  ```bash
  microk8s kubectl get pods -A -o wide
  ```

## 실습 2: 다양한 오브젝트를 사용하여 앱 배포

### Pod 오브젝트를 사용하여 앱 배포
```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: nginx 
spec: 
  containers: 
  - name: nginx 
    image: nginx:1.14.2 
    ports: 
    - containerPort: 80
```
Pod 생성 후 다음 명령어로 확인합니다:
```bash
kubectl get pods -A -o wide
```
만약 파드가 `Pending` 상태라면 다음 명령어로 상태를 확인합니다:
```bash
kubectl describe pod <pod-name>
```
Pod 삭제:
```bash
kubectl delete -f pods.yaml
```

### ReplicaSet 오브젝트를 사용하여 앱 배포
```yaml
apiVersion: apps/v1 
kind: ReplicaSet 
metadata: 
  name: replica-test 
  labels: 
    tier: replica-test 
spec: 
  replicas: 3 
  selector: 
    matchLabels: 
      tier: replica-test 
  template: 
    metadata: 
      labels: 
        tier: replica-test 
    spec: 
      containers: 
      - name: nginx 
        image: nginx
```
ReplicaSet 생성 후 확인:
```bash
kubectl apply -f replicaset.yaml
kubectl get pods -o wide
kubectl get replicaset -o wide
```

### Deployment 오브젝트를 사용하여 앱 배포
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
Deployment 생성 후 확인:
```bash
kubectl apply -f deployment.yaml
kubectl get pods -o wide
kubectl get deployment
kubectl describe deployment
```
스케일링:
```bash
kubectl scale deployment/nginx-deployment --replicas=5
kubectl get pods
```
Pod 삭제 후 확인:
```bash
kubectl delete pod <pod-name>
kubectl delete -f deployment.yaml
```

### DaemonSet 오브젝트를 사용하여 앱 배포
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
```
DaemonSet 생성 후 확인:
```bash
kubectl apply -f daemonset.yaml
kubectl get pods -o wide
kubectl get daemonset
kubectl delete -f daemonset.yaml
```

### Job 오브젝트를 사용하여 앱 배포
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: alpine
        command: ["echo", "-n", "helloooooo"]
      restartPolicy: Never
  backoffLimit: 4
```
Job 생성 후 확인:
```bash
kubectl apply -f job.yaml
kubectl get pods -o wide
kubectl get jobs -o wide
kubectl logs <pod-name>
kubectl delete pod <job-pod-name>
kubectl delete -f job.yaml
```

### CronJob 오브젝트를 사용하여 앱 배포
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
CronJob 생성 후 확인:
```bash
kubectl apply -f cronjob.yaml
kubectl get pods -o wide
kubectl get cronjob -o wide
kubectl get jobs
kubectl delete -f cronjob.yaml
```

이 실습을 통해 Kubernetes에서 다양한 오브젝트를 사용하여 애플리케이션을 배포하고 관리하는 방법을 배웠습니다.
