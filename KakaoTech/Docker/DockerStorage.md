컨테이너 내부에 생성된 모든 파일은 쓰기 가능한 컨테이너 레이어에 저장
- 컨테이너가 존재하지 않으면 데이터가 지속되지 않고, 다른 프로세스가 해당 데이터를 꺼낼 수 없다.
- 컨테이너의 쓰기 가능한 레이어는 컨테이너가 실행되고 있는 호스트 머신에 강하게 결합되어 다른곳으로 이동x
- 컨테이너의 쓰기 가능한 레이어에 쓰기를 하려면 파일 시스템을 관리하는 저장소 드라이버가 필요한데 이것은 Linux의 union file system에서 제공 -> 직접 호스트의 파일 시스템을 사용하는 것보단 성능 x

Docker의 컨테이너가 호스트 머신에 메모리 파일을 저장 할 수 있는 두가지
1. 볼륨(Volumes) 
	- 저장되는 위치 Linux - /var/lib/docker/volumes/
2. 바인드 마운트( Bind mounts)
	- 저장되는 위치는 지정한 해당 폴더

3. tmpfs 마운트: 지속되진 않지만 실행되는 동안 호스트가 저장
	- 저장되는 위치 - 호스트 시스템 메모리(파일시스템에 기록x)
4. Named pipes 

#### 볼륨 Volumes
: docker에 의해 생성되고 관리됨, /var/lib/docker/volumes/ 이 하위 폴더에서 볼륨을 관리
생성 방법
- `docker volume create <name>` : 명시적 생성
- Dockerfile에서 VOLUMES로 특정 폴더 지정(이름을 따로 지정하지 않으면 익명 볼륨으로 생성된다)
컨테이너에 마운트
- `docker run test -v <name>:/app/data/ containerimg` : -v 는 좀 간단한편
- `docker run test --mount source=<name>, target=/app/data img`: --mount는 좀 더 상세하게 표현
	- 읽기 전용으로 사용하기 `docker run test --mount source=nginx, destination=/usr, readonly`
조회 
- `docker volume ls`  - 해당 폴더 위치에 직접 접근해서 볼수도 있다.
- `docker volume inspect <name>`: 볼륨 정보 확인

제거 방법 
- `docker volume rm <name>`: 삭제
- `docker volume prune`: 사용하지 않는 모든 볼륨 삭제

주의사항
- 볼륨은 컨테이너가 생성될때 만들어지지만 컨테이너가 사라져도 볼륨은 남아있기에 관리가 필요
	- 볼륨 사용 때 `--rm` 플래그를 사용하여 컨테이너가 사라질때 같이 사라질수 있도록 관리
- 익명 볼륨은 재사용 불가능

잘 사용하는 사례
- 여러 실행 중인 컨테이너 간의 데이터 공유
- Docker 호스트에 특정 디렉토리나 파일 구조가 보장되지 않을 때
- Docker 호스트에서 다른 호스트로 데이터 백업, 복원, 마이그레이션 할 때
- 애플리케이션이 고성능 I/O를 요구할때
- 애플리케이션이 완전한 네이티브 파일 시스템 동작을 요구할 때 -> 데이터 베이스 엔진은 디스크 플러싱을 정확히 제어하여 트랜잭션 내구성을 보장
- 이렇게도 사용 가능


#### 바인드 마운트 Bind Mounts
: 호스트 머신의 파일이나 디렉토리가 컨테이너에 마운트 됨. 파일이나 디렉토리는 호스트 머신의 경체 경로로 참조된다. 

사용 방법
- `docker run -d --name mycontainer -v /path/on/host:/path/in/container myimage`
	- 볼륨과 차이점 볼륨을 사용할땐 <볼륨 이름>:<컨테이너 경로> 였는데 바인드 마운트는 <호스트 경로>:<컨테이너 경로>로 사용
- `docker run -d --name mycontainer --mount type=bind,source=/path/on/host,target=/path/in/container myimage`
	- --mount로는 type=bind로 명시적 사용
- 읽기 전용
	- `docker run -d --name mycontainer -v /path/on/host:/path/in/container:ro myimage`

잘 사용하는 사례
- 호스트 머신에서 컨테이너로 구성 파일을 공유 할 때
- Docker 호스트의 개발환경과 컨테이너 간에 소스 코드나 빌드 산출물을 공유 할 때
	- 이걸 이용하면 깃허브를 통해서만 빌드된 것을 서버에 적용시키는 것이 아닌 컨테이너 한쪽에서는 개발하고 다른 한쪽에서는 테스트 하게 활용가능 할듯(간단한 정도에서만)
- Docker 호스트 파일이나 디렉토리 구조를 그대로 사용하여 컨테이너 안에서의 파일 구조를 일관적으로 들고 있게할 때


#### tmpfs 마운트 
- Docker 호스트나 컨테이너 내에서의 디스크에 저장x 

사용 방법
- `docker run -d --name mycontainer --tmpfs /app:rw,size=100m myimage`
- `docker run -d --name mycontainer --mount type=tmpfs,destination=/app,tmpfs-size=100m,tmpfs-mode=1777 myimage`
	- 옵션
		- 크기 제한(size /  tmpfs-size) : tmpfs 마운트의 최대 크기 지정 
		- 액세스 모드(mode / tmpfs-mode): tmpfs 마운트의 파일 권한 설정 ex. mode=1777

잘 사용하는 사례
- 컨테이너가 지속되지 않는 상태가 민감한 정보를 저장하는데 사용

#### + Named pipes
- Docker 호스트와 컨테이너 간의 통신에 사용 할수도 있다.
- 컨테이너 내부에서 서드파티 도구를 실행하고 이 파이프를 통해 도커 엔진 api에 연결


#### Storage Driver




Q. **docker storage 에서 명령어를 쓰는 것과 도커 파일에서 작성하는 볼륨은 다른가?** 
- 아니? 같다 ==== 다 똑같이 생성된다(명명 볼륨,익명볼륨 차이)

**Q. 보통 데이터베이스 인스턴스는 도커 볼륨을 활용하여 성능을 끌어 올리는편인지?**
볼륨은 Linux VM에 저장되므로 이러한 보장을 할 수 있지만, 바인드 마운트는 macOS나 Windows에 원격으로 연결되므로 파일 시스템이 약간 다르게 동작합니다.라고 해서 

**Q. 도커 관련으로 먹잇감 던져주기 가능한가요?**
- nginx파드랑 앱 두개로 무중단 배포해보기

**Q. 도커의 스토리지 드라이버는 도커의 스토리지 기능을 쓰기 위해 어떤 파일 시스템을 쓸것인지에 대한 건가요?**
- 네 맞습니다. 기존에는 리눅스 커널이 업데이트 되기전에 거의 aufs를 사용했었는데 overlay2라는 스토리지 드라이버가 등장하면서 현재 기준 대부분의 커널에서는aufs를 만료시켰다 라고 보시면될것같아요
- zfs도 쓰는편? >> zfs는 좀 많이 먼 이야기

