#### Docker compose
: 여러가지 도커 컨테이너르 하나의 애플리케이션으로 정의하고 관리하기 위한 도구
: 여러개의 도커 이미지를 하나의 스크립트에서 다 할 수 있음 > 프론트, 백엔드 등등

안에 도커 파일이 있으면 실행되는 것처럼 
- build
	- context: 도커 파일이 있는 디렉토리를 지정 
	- dockerfile: 사용 할 dockerfile 이름(경로)를 지정
- image
	- 이미지를 빌드하고 가져올 수 있음
- ports `"5000:5000"`
	- 호스트와 컨테이너 간의 포트 매핑을 지정
- environtment
	- 컨테이너 내에서 사용 할  환경 변수를 지정
	- `FLASK_ENV=development`
- volumes
	- 호스트와 컨테이너 간의 파일 시스템 마운트 지정
	- 다양한 이유에 의해 쓰게 됨 > 직접 실습하면서 마운트 되었는지 확인해보자
- depens_on
	- 컨테이너 간의 의존성을 지정
	- 어떤 서비스와 의존하겠다 선언
- networks
	- 도커를 사용하게 되면 네트워크 상에 가상ip가 할당되는데 > 서비스하는 단계에서 계속 통신해야 하기 때문에 계속 사용 > 특정 ip 지정하게끔? 특정 범위를 지정?
- restart
	- 어떤 장애를 만났을때 재시작하게끔 설정 할 수 있다. 
	- always/on-failuere/
- volumes(전역)
- networks(전역)

#### Docker compose 기능
- 멀티 컨테이너 애플리케이션 정의
	- yaml 은 json과 달리 주석이 된다
	- docker-compose.yml 파일을 통해 여러 컨테이너를 정의
	- yaml 문법에 대해 공부 필요
- 서비스 간 의존성 관리
	- depens on을 써도되고 안써도 되고
- 일관된 개발 환경 제공
	- 동일한 yml을 사용하면 로컬인지 프로덕션인지 일관되게 설정
	- 개발과 운영 간 환경 차이를 많이 줄여줌
- 간편한 실행 및 종료


이렇게 길게 쓸수도 있다~
```
docker run --rm -it -v  /proc:/proc -p 5000:5000  —pid=host --cap-add=SYS_PTRACE ubuntu /bin/bash
```

#### Docker compse 주요 명령어
- docker compose up: yml 파일을 실행시키겠다는 뜻
- docker compose down: 실행중인 모든 서비스 중지, 네트워크, 볼륨을 포함한 모든 리소스 정리
- docker-compse ps: 목록
- docker compse logs: 로그 출력
