#### Kubernetes
- cluster : master 1대 / worker node 2대
- control plane == master node
- worker load == node
- worker load, master node의 컴포넌트 구조가 각각 다르게 이뤄져 있다. 

- pod: 쿠버네티스에서 배포할 수 있는 가장 작은 단위
	- 하나 이상의 컨테이너 그룹
- replicaset : 
	-  외부에 트래픽이 몰리면 끝 pod에 가게되는데 한곳에 몰리게 하는것이 아니라 여러곳으로 트래픽을 분산
	- 자동으로 스케일 업
	- 안정성있게 최대한 가용성을 보장
	- 요즘은 deployment를 함께 쓰는 추세
- deployment
	- 죽어도 레플리카 개수만큼 다시 생겨남
	- => 서비스가 죽어도 pod가 계속 생겨난다
- statefulset
	- pod는 stateless가 일컫는데, pod는 해당 이미지에 대한 상태만 가지고 있고 서비스에 대한 상태를 갖고 있지 않음
	- statefule를 관리하는데 사용
	- 상태를 저장하냐 저장하지 않냐의 차이
- Daemonset - 이거 뭐지
- Job / CronJob > 주로 크론잡을 
	- 주기적으로 액션해야 할때 ex. 알람, 주기적으로 실행 해야할 떄 그때 보통 cron을 한다? 라고 하는데 > 어떠한 명령어라든지, 앱을 주기적으로 실행시키려할때
	- job 한번만 실행? 파드를 실행 > pod가 성공적으로 될때까지 실행
	- cronjob > 이거는 job을 실행시키는 역할 
		- corntab 파일 한 줄과 같음

#### Kubernetes Object
- namespace
	: 리소스를 논리적으로 분리
	- default 
	- kube-system
	- kube-public

#### Kubectl
- 클러스터란? 관리할때 몇천대씩 있을때 클러스터 단위로 처리하게 되는데, master node가 n개, workerload가 n개로 구성된 그룹을 클러스터
- 한 클러스터를 aws, gcp, on-premise 등등 다양한 곳에서 클러스터를 스위치하면서 조회하는 방법으로 어떤 클러스터가 있는지 `kubectl cluster-info`

자주 쓰는 명령어
- exec: 배포된 pod를 어떻게 실행
- run: 배포된 pod를 생성? 
- expose : 외부에서 접근할때
- port-forward: 클러스터 내 오브젝트에 붙기 위해
- kubeconfig: 어떨때는 api로 던지고 하여 여러개의 클러스터를 한곳에서 쓰려고
- 쿠버네티스는 전부 다 api로 통신한다 생각하면 편함 > 마스터 노드의 api 서버를 통해 각 pod에 전달된다는 느낌 > **"환경에 제약없이 api 콜만 되면 된다."**
  클러스터를 올리고 pod 를 올리는데만 리눅스에서 사용하고 호출하는 것은 어디든 괜찮다.
