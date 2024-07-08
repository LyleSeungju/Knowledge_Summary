## Linux 심화 

#### Cgroup. Control Group
: 커널에서 제공하는 기능 / 프로세스 그룹에 대한 시스템 리소스 사용을 관리(제한)
: 특정 프로세스 그룹이 사용 할 수 있는 것(CPU, 메모리, 디스크i/o,네트워크 대역폭)들을 제한하거나 할당

기능
- 리소스 제한: 프로세스가 사용할 수 있는 자원을 제한 > 시스템 전체의 안정성 보장
- 리소스 우선순위: 중요한 프로세스에 더 많은 자원 / 덜 중요? 덜 자원
- 리소스 계정: 프로세스 그룹 별 자원 사용량 모니터링 & 통계
- 리소스 격리: 프로세스 그룹 간 자원 사용 격리 > 다른 그룹에 영향 x
구조(주요 3가지)
- cpu: cpu 제한 또는 보장
- memory: 메모리 사용 제한 또는 사용 통계 제공
- blkio: 블록 i/o 사용량을 제한하고 모니터링
##### 실습

cpu제한 실습
- sudo cgcreate -g cpu: /cpulimit
	- cgcreate: cgroup(컨트롤 그룹) 생성
	- -g : cgroup(컨트롤 그룹) 경로 지정
- echo 50000 | sudo tee /sys/fs/cgroup/cpulimit/cpu.max
	-  /sys/fs/cgroup/cpulimit/cpu.max 에 작성
	- cpu 사용량 50%퍼로 줄임 > 100ms 주기동안 50ms만 실행 가능
- sudo cgexec -g cpu: /cpulimit stress --cpu 2 --vm 1 --timeout 30s &
	- cgexec: cgroup 실행 명령어
	- &: 백그라운드 실행
	- --cpu 2 : 2코어 사용
	- --vm 1 : 메모리 사용하는 1개의 작업
	- -timeout 30s : 30초 뒤 자동 종료
- top
	- 메모리 사용량, CPU 사용량 등을 나타내주는 명령어
- sudo cgdelete -g cpu: /cpulimit
	- cpu 제한 삭제

메모리 제한 실습
- sudo cgcreate -g memory:/memorylimit
- echo $((512 * 1024 * 1024)) | sudo tee /sys/fs/cgroup/memorylimit/memory.max
	- 메모리 사용량 512mb 제한 걸기
- sudo cgexec -g memory:/memorylimit stress --vm-bytes 1024M --vm 1 --timeout 30s & 
	- –vm-bytes 1024M: 메모리 작업이 1기가 부하 걸기
	- 30초동안 작업 실행 > 512mb로 제한을 걸었기 때문에 512까지 실행이 안됨
	- 백그라운드로
- watch -n 0.5 cat /sys/fs/cgroup/memorylimit/memory.current
	- 실시간 메모리 사용량 보기
	- 0.5초 간격으로 모니터링
- sudo cgdelete -g memory:/memorylimit
	- 메모리 제한 삭제


#### Namespace 
: 주로 컨테이너에서 사용됨
: 프로세스가 시스템 리소스를 격리하여 사용 가능하게 하는 커널 기능
: 시스템 자원 간 충돌 방지 / 프로세스나 프로세스 그룹이 독립된 환경에서 실행

- 종류
	- **mount namespace(mt): 파일시스템 네임스페이스**
		- 네임시스템마다 파일시스템을 가지고 있고 서로 볼 수 없음
	- **process ID namespace(pid): 프로세스 id 격리하고 독립된 프로세스 id 공간을 가짐**
		- 네임 스페이스 내에 pid 1로 새로 격리하여 사용
		- 독립적인 PID만 제공됨
			- 모두 각각 개별의 실습이고 그걸 다 합쳐야 컨테이너 관련된 격리 기술이 나오게 됨
			- 프로세스 계의 chroot같은 개념
			- 새로 뜬 pid를 새로운 네임스페이스에서 1번 root process로 제공함
			- == host의 5196번 = pid namespace 1번
		- 지피티한테 정리
			1. **독립된 PID 공간**: 각 PID 네임스페이스는 자신만의 프로세스 ID 공간을 가지며, 네임스페이스 내의 프로세스들은 다른 네임스페이스의 프로세스를 볼 수 없습니다.
			2. **init 프로세스**: 새로운 PID 네임스페이스가 생성되면, 그 안에서 첫 번째 프로세스는 항상 PID 1을 가지며, 이는 해당 네임스페이스의 init 프로세스로 동작합니다.
				- ex. 예: 호스트의 PID 5196번 프로세스는 PID 네임스페이스 내에서는 1번 프로세스로 동작합니다.
			3. **컨테이너 격리 기술**: PID 네임스페이스는 다른 네임스페이스(예: 마운트 네임스페이스, 네트워크 네임스페이스 등)와 결합하여 컨테이너의 격리 기술을 완성합니다. 이는 프로세스, 네트워크, 파일 시스템 등 다양한 자원을 격리하여 독립적인 환경을 제공합니다.
	- **network namespace(net): 네트워크 장치, ip 주소, 포트 번호 등을 격리**
		- netstat을 사용하여 실습할 수도 있음
		- 네임 스페이스 간에 네트워크 장치를 독립적으로 설정
	- **user namespace(user) : 사용자 / 그룹 ID 네임스페이스**
		- 사용자,그룹 id 등 격리 / 권한 관리
		- 컨테이너 안에서는 루트 권한을 가지지만, 호스트 시스템에서는 루트 권한을 가지지 않음
		- > 시스템 보안을 유지하며 각 컨테이너가 독립적으로 루트 권한을 가짐
	- **IPC namespace(ipc) : 프로세스 간 통신 네임 스페이스**
		- 각 네임스페이스는 독립된 IPC 공간을 가지므로, 다른 스페이스와 충돌없이 **메세지 큐, 세마포어, 메모리 공유** 사용
	- **UTS namespace(uts) : 호스트 이름, 도메인 이름 격리 네임 스페이스**
		- 각 네임스페이스는 독립된 호스트 이름과 도메인 이름을 가질 수 있다. 
		- > 컨테이너화된 환경에서 시스템 식별 정보를 별도로 설정하는데 필요

	**Mount Namespace 실습 - <파일 시스템 네임 스페이스>**
	- sudo unshare --mount /bin/bash
		- 새로운 마운트 네임스페이스를 생성하고, 그 안에 있는 배쉬 셸을 실행
	- mkdir /mnt/test
	- mount -t tmpfs none /mnt/test
		- 임시 파일 시스템을 붙여줌
	- echo "Hello from Mount Namespace" > /mnt/test/hello.txt
		- 새로운 파일 생성
	- cat /mnt/test/hello.txt
		- 내용
	- exit
	- ls /mnt/test 
		- 기존 환경에서 확인하면 분리된 네임스페이스에서 존재하는 파일은 이미 종료되어 확인되지 않는다. 
	**PID namespace 실습 - <프로세스 네임 스페이스>**
	 sudo unshare --pid --fork /bin/bash 
	 - ps -e  
	- `echo $$`
		- 분리된 네임스페이스에서는 기존 pid와 다르게 자신이 실행되고 있는 pid 번호 실행
	- exit
	- `ps -e echo $$`
	**User namespace 실습 - <사용자, 그룹 id네임스페이스>**
	- sudo unshare -user /bin/bash
	- id
	- touch /root_test_file
	- ls -l /root_test_file
		- 내부에선 외부에 건들지 못하지만
		- 외부에서 내부는 건들 수 있다.
	- exit
	- sudo rm /root_test_file
	**UTS namespcae 실습- <호스트네임 그룹스페이스>
	- hostname  
	- sudo unshare -uts /bin/bash
	- hostname kakao<영어이름> hostname
	- exit
	- hostname

#### Chroot 
:  change root > 루트 디렉토리 위치를 변경하는 역할
: chroot jail : 특정 루트를 설정하면 실제 그 상위에 디렉토리가 있어도 해당 파일이 루트파일이기때문에 못 빠져나온 다는 의미에 jail이라 한다.
:제한된 환경에서 테스트를 하거나 보안 목적으로 사용되기도 하지만 추후에 학습할 Docker에서도 사용되는 명령

Chroot 실습 환경 세팅
- sudo mkdir -p /mychroot/{bin,lib,lib64,dev,etc,proc,sys}
	- mychroot 밑에 일련의 폴더를 생성한다
	- -p : 자동으로 부모를 생성
- sudo cp /bin/* /mychroot/bin/
	- 내부에서 사용할 바이너리 복붙
- `for lib in $(ldd /bin/bash /bin/ls /bin/cat | grep -o '/lib[^ ]*'); do sudo cp --parents "$lib" /mychroot; done
	- ldd : 지정된 실행 파일의 공유 라이브러리 의존성 출력
	- '/lib[^ ]*' : 정규 표현식인데 lib으로 시작해서 스페이스를 만나기전까지, 모든 lib으로 시작하는 글자 출력
	- for lib in : 추출된 경로를 순회하는 루프
	- sudo cp –parents “$lib” /mychroot: 각 라이브러리를 /mychroot 디렉토리로 부모 디렉토리 구조를 포함하여 복사
가상 파일 시스템 마운트 : 미리 프로세스의 파일을 가져옴
- sudo mount -t proc proc /mychroot/proc
- sudo mount -t sysfs sys /mychroot/sys
- sudo mount --bind /dev /mychroot/dev
Chroot 환경 진입
- sudo chroot /mychroot /bin/bash
	- chroot로 빈배쉬를 접근
- pwd : 
- ls /
	- 아까 가져온거 그대로 사용 가능
- echo "Hello from chroot!" > /etc/kakao
- cat /etc/kakao
	- 파일 생성된거 확인
- exit

환경 정리
- sudo umount /mychroot/proc
- sudo umount /mychroot/sys
- sudo umount /mychroot/dev
- sudo rm -rf /mychroot

#### Unix Domain Socket. UDS
: 동일 호스트 내에서 **프로세스 간 통신**을 위한 매커니즘
: 네트워크를 통해 통신하는 것이 아닌 로컬 파일 시스템을 통해 통신
- 로컬 통신 : 네트워크로 데이터를 보내는게 아니기때문에 더 빠름
- 파일 시스템을 이용한 주소 지정: 파일 시스템 경로를 사용하여 식별 
- 데이터 전송 속도: 네트워크 계층을 거치지 않아서 더 빠름
- 보안: 파일 시스템의 권한 설정으로 접근 제어 > 특정 사용자나 프로세스만 소켓에 접근하게끔 가능



#### OverlayFS
: 오버레이 파일 시스템
 ~~20.04까지 오버레이+uts가 동시에 제공되었지만 22.04 부터는 uts만~~
: 파일시스템을 제공해주고, 파일 시스템을 관리 할 수 있도록 각각 디렉토리를 나누어 저장

**??? 쓰는 이유가 뭐야? 
- 효율적인 스토리지 사용
	- lowerdir: 변경되지 않는 기본 파일 시스템 / 여러 컨테이너나 사용자들이 동일 이미지 사용 가능
	- upperdir: 각 컨테이너나 사용자가 자신만의 변경 사항 저장 & 독립적인 파일 시스템 가능
- 빠른 파일 시스템 복사
	- 카피 온 라이트(copy-on-write): 기존 파일을 수정하지 않고, 수정된 부분만 uppderdirdp 저장
- 컨테이너 환경에서의 유용성
	- 이미지 레이어링: 새로운 이미지를 생성할때 변경된 부분만 새로운 레이어로 추가
	- 격리: 컨테이너마다 독립적인 uppderdir 사용 > 서로 영향 x
- 간단한 파일 시스템 스냅샷: 현재 상태의 파일 시스템은 스냅샷으로 저장, 이후 변경 사항만 uppderdir에 저장하여 백업과 복원 간편 > lowerdir가 초기 스냅샷이라 생각하면 될거같음

**주요 특징
- 파일 시스템 계층화 : 읽기 전용 & 쓰기 가능한 계층을 병합 사용
- 스냅샷 및 버젼 관리: 파일 시스템의 상태를 쉽게 스냅샷으로 저장하고, 필요 시 복원
- 컨테이너: 도커와 같은 곳에서 이미지 레이어링을 위해 사숑
**파일시스템 구조**
- Lowerdir: 저장을 했을때 기반이 되는 >  불변하는 읽기 전용 시스템 
- Upperdir: 수정하는 내용
- workdir: 변경사항 추적, 없으면 정상동작 x
- Merged (lower + upper)
**작동 방식**
1. **새로운 파일 생성**: 파일은 Upperdir에 생성됩니다.
2. **기존 파일 읽기**: 파일은 Lowerdir에서 읽힙니다. Upperdir에 수정된 버전이 있으면 Upperdir에서 읽힙니다.
3. **기존 파일 수정**: 파일은 Upperdir에 복사되어 수정됩니다.
4. **파일 삭제**: Upperdir에서 파일 삭제 플래그를 설정합니다. 실제 파일은 Lowerdir에서 삭제되지 않습니다.

- sudo mkdir -p /lower /upper /work /merged
	- 4개 디렉토리 생성
- echo "Lowerdir 입니다" | sudo tee /lower/lowerfile.txt
- echo "Upperdir 입니다" | sudo tee /upper/upperfile.txt
- sudo mount -t overlay overlay -o lowerdir=/lower,upperdir=/upper,workdir=/work /merged
	- 오버레이 파일 시스템을 특정 파일에 연결
	- overlay : 오버레이 파일 시스템
	- lower + upper + work => merged 파일 시스템을 만들겠다
- ls /merged

- `>> merged에서 lowerfile.txt와 upperfile.txt 확인
- cat /merged/lowerfile.txt
- cat /merged/upperfile.txt
- echo "Mergeddir 입니다" | sudo tee /merged/meregdfile.txt
	- ls -al 
	- merged를 수정하면 upper과 merged에 기록
-  ls /upper

- sudo umount merged
- sudo rm -rf /lower /upper /merged /work

**Q: Lowerdir에 있는 파일을 수정하기 위해 Upperdir로 수동으로 복사해서 수정해야 하나요?**
• 실제로 사용하는 디렉토리는 Merged입니다. 파일을 수정하면 자동으로 Upperdir에 복사되어 수정됩니다. 수동으로 복사할 필요는 없습니다.

**Q: 수정된 파일이 Lowerdir로 복사되지는 않나요?**
• 맞습니다. Lowerdir는 불변입니다. 파일 수정 시 Upperdir에만 변경 사항이 기록되며 Lowerdir에는 영향을 미치지 않습니다.

**Q: Merged에서 파일을 가져올 때 실제 작동 방식은 Upperdir와 Lowerdir를 참조하는 건가요?**
• 맞습니다. 파일을 읽을 때는 우선 Upperdir를 참조하고, Upperdir에 파일이 없으면 Lowerdir를 참조합니다. 파일 수정은 Upperdir에만 기록됩니다.

**Q: Lowerdir는 시스템 설정 시에만 파일이 작성되고 그 이후에는 참조만 하나요?**
• 맞습니다. Lowerdir는 초기 시스템 설정 시에만 파일이 작성되고, 이후에는 읽기 전용으로 참조됩니다. 수정이나 추가는 Upperdir에서 처리됩니다.

**Q: 두 번 수정이 일어나는 상황에서는 Upperdir에 파일이 있으므로 Lowerdir에서 복사하지 않나요?**
• 맞습니다. 이미 Upperdir에 파일이 존재하면 Lowerdir에서 복사하지 않습니다.

**Q: Workdir의 변동 사항 추적과 Upperdir의 변동 사항 추적은 어떻게 다른가요?**
• Workdir는 변경 사항을 추적하는 데 사용되고, Upperdir는 실제 변경 사항을 저장합니다. Workdir는 OverlayFS의 내부적인 동작을 위한 것입니다.




