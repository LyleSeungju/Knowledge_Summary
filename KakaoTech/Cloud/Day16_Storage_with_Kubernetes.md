#### 쿠버네티스 자격증 CKA
- 공부 방법 
	- https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/
	- [인프런 무료강의](https://www.inflearn.com/course/%EA%B3%B5%EC%9D%B8-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EA%B4%80%EB%A6%AC%EC%9E%90#curriculum)

- [시험 커리큘럼](https://github.com/cncf/curriculum/blob/master/CKS_Curriculum_%20v1.30.pdf)
- 리눅스 커맨드 문제 해결하면서 진행
- 15~20개, 2시간
- 시험일때 주어지는 것은 [공식문서](https://kubernetes.io/docs/home/) 하나 뿐임
- `dry-run` : 명령어 자주 썼음


#### Kubernetes Storage
- hostpath
	- readonly 만 해서 읽게만 시키는 방법
	- 디버깅용으로 쓰는 방법(상태 리스이기 때문에)
- emptyDir
	- 비어 있는 디렉토리를 만듦
- Configmap : 가장 많이 사용
	- 환경변수의 주입
	- 일일히 파드마다 관리하는 것이 아닌 이걸로 재사용성 있게 쓸 수 있음
- Secret : 비밀이지만 인코딩 되어 있다.
	- 민감한 정보 > 진짜 민감한것은 좀 더 암호화
	- 각각의 특징도 있음
- PV
	- 큰 박스
	- 동적
- PVC
	- pv에서 필요한만큼 쪼개서 사용

#### 쿠버네티스 상태 관리

Probe
- kublelet(파드 상태 관리) > 주기적으로 컨테이너 상태를 체크 > 문제가 있는 컨테이너를 재시작, 서비스에서 제외
Handler
- ExecAction - 실행 느낌
- TCPAction - tcp 소켓 연결 시도하며 체크
- HTTPGetAction - 특정 url로 http get 요청을 하며 200~400이 아닌 경우 실패
Probe 종류
- Liveness Probe
	- 애플리케이션 상태를 체크해서 서버가 제대로 
	- 사용 : 컨테이너 속 프로세스가 중단되면, kubelet 의 pod의 restartPolicy에 따라 자동으로 수행
- Readiness Probe
	- 컨테이너가 요청을 처리 할 준비가 되었는지 확인
- Startup Probe
	- 잘 실행이 되는지 확인 
	- startup 성공해야만 liveness, readiness 가 동작
-> 각각의 프로브가 어디에 써야한다는 정답은 없다. 필요하다고 생각하는 것을 해보는 것

## 실습 1

- hostPath
    - hostPath 볼륨을 사용하여 파드에 파일 생성 및 확인
        - 환경 세팅 : master 1, worker 1 구성
        - 파일 세팅 : yaml 파일에 hostpath 항목 확인
        - 폴더 세팅 : 배포 전 각 노드에 mkdir data로 호스트에 data 폴더 생성 - 어떤 노드에 생성될지 모르니까
            - 실습할땐 `/data` 경로로 폴더 생성
            - 일반적으로 파일을 어떤 경로에 마운트 하나요?
                - 일반적으로 마운트하면 `/mnt` 폴더
                - 보안적으로 위험한 경우 `/tmp` 폴더
            
        
        ```bash
        k apply -f hostpath.yaml
        
        k get pods -o wide #어떤 노드에 생성되었는지 확인
        
        k exec -it test-pd -- /bin/bash #k exec test-pd -- cat /test-pd/kakao
        
        cd /test-pd
        echo "kakao" > kakao
        
        cd /data  #노드가 있는 호스트에 동일한 파일이 있는 것을 확인
        
        # hostpath.yaml를 아무리 지워도 호스트와 마운트 되어 있다면 해당 파일이 있는것을 볼 수 있다.
        k delete -f ~.yaml
        ```
        
    - (추가 실습) 두개의 파드가 하나의 hostpath볼륨을 공유
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: test-pd-1
        spec:
          containers:
          - image: nginx
            name: test-container-1
            volumeMounts:
            - mountPath: /test-pd
              name: test-volume
          volumes:
          - name: test-volume
            hostPath:
              path: /data
              type: Directory
        ```
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: test-pd-2
        spec:
          containers:
          - image: nginx
            name: test-container-2
            volumeMounts:
            - mountPath: /test-pd
              name: test-volume
          volumes:
          - name: test-volume
            hostPath:
              path: /data
              type: Directory
        ```
        
        1. 두개의 파드 생성 후 첫번째 파드에서 /test-pd 경로에 새로운 파일 생성
        2. 해당 노드의 호스트와 두번째 파드에서 생성된 파일 확인
- emptyDir
    - emptyDir 볼륨을 사용해보기
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: test-pd
        spec:
          containers:
          - image: nginx
            name: test-container
            volumeMounts:
            - mountPath: /cache
              name: cache-volume
          volumes:
          - name: cache-volume
            emptyDir:
              sizeLimit: 500Mi
        ```
        
        1. 파드 생성 
        2. 파드 상태 확인
        3. 생성된 파드에 접속
        4. emptyDir 볼륨에 데이터 저장
            
            `echo "Hello, EmptyDir!" > /cache/test-file`  : 파일 생성된 것을 확인
            
        5. 접속을 종료하고 해당 파드 삭제
        6. 다시 파드를 생성해서 접속해보면 해당 파드의 emptyDir에 데이터가 사라졌음을 알 수 있다.
            
            
    - (추가 실습) 여러 컨테이너에서 같은 emptyDir볼륨을 공유해보기
        
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: multi-container-pod
        spec:
          containers:
          - name: nginx-container
            image: nginx
            volumeMounts:
            - mountPath: /cache
              name: cache-volume
          - name: busybox-container
            image: busybox
            command: ['sh', '-c', 'while true; do sleep 3600; done']
            volumeMounts:
            - mountPath: /cache
              name: cache-volume
          volumes:
          - name: cache-volume
            emptyDir: {}
        ```
        
        1. emptyDir-multi.yaml 파일 작성
        2. 파드 생성
        3. 파드 상태 확인
        4. nginx-container 에 접속하여 `/cache` 폴더에 새로운 데이터 생성 
            
            `k exec -it multi-container-pod -c nginx-container -- /bin/bash`
            
        5. busybox-container에 접속하여 `/cache` 폴더에 파일 존재 확인
            
            `k exec -it multi-container-pod -c busybox-container -- /bin/sh`
            
            `ls /cache`
            
        6. 마운트된 해당 볼륨에 같은 데이터를 공유하고 있는 것을 볼 수 있다.
- PV / PVC
    - PV 생성 및 PVC 할당해보기
        
        ```yaml
        # pv.yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: pv-example
        spec:
          capacity:
            storage: 1Gi
          accessModes:
            - ReadWriteOnce
          persistentVolumeReclaimPolicy: Retain
          hostPath:
            path: "/mnt/data"
        ```
        
        ```yaml
        #pvc.yaml
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: pvc-example
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 750Mi
        ```
        
        1. pv 생성
        2. `k get pv` : 상태확인
        3. pvc 생성
        4. `k get pvc` : 상태확인
            - 여기서 보이는 Capacity는 pvc의 capacity가 아닌 pv의 capacity이기 때문에 실제로는 750Mi만 할당되었더라도 표시는 pv를 따라가게 되어 1Gi 로 표시된다.
            - 남은 250Mi에 할당하려고 해도 작동이 안됨 → 하나의 PV에는 하나의 PVC만 할당 가능
        5. 만약 PVC만 삭제하고 다시 PV에 마운트 하려면
        `k patch pv [pv name]-p '{"spec":{"claimRef": null}}'` 를 다시 해줘야 한다.
    - ****여러 PVC를 할당 (동적 프로비저닝 사용)
        
        ```yaml
        # storageClass.yaml 스토리지 클래스
        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: standard
        provisioner: kubernetes.io/no-provisioner
        volumeBindingMode: WaitForFirstConsumer
        ```
        
        ```yaml
        #동적 프로비저닝 사용하는 PVC 정의
        #pvc-dynamic1.yaml
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: pvc-dynamic-example-1
        spec:
          storageClassName: standard
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
        #pvc-dynamic2.yaml
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: pvc-dynamic-example-2
        spec:
          storageClassName: standard
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
        ```
        
        1. 스토리지 클래스 생성
        2. pvc1 생성 후 확인 `k get pvc`
        3. pvc2 생성 후 확인
        4. 해보는중 → 실패하는중 

## 실습 2

- configmap
    - configmap 사용해서 저장된 정보 읽기 1
        
        ```yaml
        #configmap1.yaml
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: myapp-config
          namespace: default
        data:
          APP_GREETING: "Hello"
          APP_NAME: "World"
        ```
        
        ```yaml
        #configmap1_pod.yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: test-pod-env
        spec:
          containers:
          - name: test-container
            image: busybox
            command: ['sh', '-c', 'echo $(APP_GREETING) $(APP_NAME) && sleep 3600']
            env:
            - name: APP_GREETING
              valueFrom:
                configMapKeyRef:
                  name: myapp-config
                  key: APP_GREETING
            - name: APP_NAME
              valueFrom:
                configMapKeyRef:
                  name: myapp-config
                  key: APP_NAME
        ```
        
        1. configmap apply
        2. configmap_pod apply
        3. 파드 상태 확인
        4. 파드 로그 확인
        5. 파드 접속해서 `env` 해보면 
            
            APP_GREETING=Hello
            
            APP_NAME=World
            
            인거 확인
            
        6. 종료
    - configmap 사용해서 저장된 정보 읽기 2
        
        ```yaml
        #configmap2.yaml
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: nginx-config
          namespace: default
        data:
          nginx.conf: |
            events {
              worker_connections 1024;
            }
            http {
              server {
                listen 80;
                location / {
                  root /usr/share/nginx/html;
                  index index.html index.htm;
                }
              }
            }
          index.html: |
            <!DOCTYPE html>
            <html>
            <head>
                <title>Welcome to Nginx!</title>
            </head>
            <body>
                <h1>Success! The Nginx server is working!</h1>
            </body>
            </html>
        ```
        
        ```yaml
        #configmap2_pod.yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: test-pod-nginx
        spec:
          containers:
          - name: nginx
            image: nginx
            volumeMounts:
            - name: config-volume
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
            - name: html-volume
              mountPath: /usr/share/nginx/html/index.html
              subPath: index.html
          volumes:
          - name: config-volume
            configMap:
              name: nginx-config
          - name: html-volume
            configMap:
              name: nginx-config
        ```
        
        1. 파드 생성
        2. 파드 상태 확인
        3. `k exec -it test-pod-nginx -- curl [localhost](http://localhost)`  : nginx 정상 실행 확인 
        4. 이미지가 어떤 환경인지 알고 싶다? `k get configmap <configmap-name> -o yaml`
- secret
    - secret 볼륨을 사용해보기
        
        ```yaml
        #secret.yaml
        apiVersion: v1
        kind: Secret
        metadata:
          name: myapp-secret
          namespace: default
        data:
          username: YWRtaW4=  # base64 encoded value of 'admin'
          password: cGFzc3dvcmQ=  # base64 encoded value of 'password'
        ```
        
        1. 생성
        2. `kubectl apply -f secret.yaml`
        3. `kubectl get secret myapp-secret -o yaml` 하면 나오는 값 들 중 
        password와 username에 ~=로 끝나는 것 복사하기
        4. `echo 'YWRtaW4=' | base64 --decode`  복사한 것을 ‘’에 넣으면 원래 값을 디코딩해볼 수 있다
        5. 다음 시크릿 실습 할거면 삭제x
    - secret 볼륨을 쓰는 파드를 생성해서 사용되는 로드 확인
        
        ```yaml
        #secret_pod.yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: secret-test-pod
        spec:
          containers:
          - name: test-container
            image: busybox
            command: ['sh', '-c', 'echo $(USERNAME) $(PASSWORD) && sleep 3600']
            env:
            - name: USERNAME
              valueFrom:
                secretKeyRef:
                  name: myapp-secret
                  key: username
            - name: PASSWORD
              valueFrom:
                secretKeyRef:
                  name: myapp-secret
                  key: password
        ```
        
        1. 생성
        2. 파드 상태 확인
        3. `k logs secret-test-pod` : ~ 아 이런게 뜨는구나
- envFrom 사용 > 여러키를 한번에 환경 변수로 사용 가능
    - configmap에서의 envfrom
        
        ```yaml
        # 그대로
        # configmap.yaml
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: myapp-config # 해당 configmap 이름 
          namespace: default
        data:
          APP_GREETING: "Hello"
          APP_NAME: "World"
          
        # configmap_pod.yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: config-envfrom-pod
        spec:
          containers:
          - name: test-container
            image: busybox
            command: ['sh', '-c', 'echo $APP_GREETING $APP_NAME && sleep 3600']
            envFrom:
            - configMapRef:
                name: myapp-config # 아까 선언한 configmap 정보 그대로 사용
        ```
        
    - secret에서의 envfrom
        
        ```yaml
        # 그대로
        # secret.yaml
        apiVersion: v1
        kind: Secret
        metadata:
          name: myapp-secret # secret 키 이름
          namespace: default
        data:
          username: YWRtaW4=  # base64 encoded value of 'admin'
          password: cGFzc3dvcmQ=  # base64 encoded value of 'password'
          
        # secret_pod.yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: secret-envfrom-pod
        spec:
          containers:
          - name: test-container
            image: busybox
            command: ['sh', '-c', 'echo $username $password && sleep 3600']
            # 여기만 추가 되는 부분
            envFrom:
            - secretRef:
                name: myapp-secret # secret 선언해놓은거 그대로 사용한다는 뜻
        ```
