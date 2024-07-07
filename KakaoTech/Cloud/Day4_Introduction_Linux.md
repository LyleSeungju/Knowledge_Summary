# Linux 기초

#### 오픈소스의 개념 OSS
: 소스 코드를 누구나 열람, 수정 배포 할 수 있는 소프트웨어
- 투명성
- 협업과 혁신
- 비용 절감 
- 자유와 통제

#### GPL License
- GNU General Public License(GPL): 자유 소프트웨어 재단에서 만든 라이선스
- 소프트웨어를 자유롭게 사용 할 수 있고, 소스 코드 를 수정 및 배포할 수 있는 권리 보장

#### 결론
- 오픈소스 커뮤니티와 자유 소프트웨어 운동을 통해 리눅스가 발전되었다

### Linux 구조
- 애플리케이션
- 쉘
- 커널
- 하드웨어

#### 애플리케이션
: 사용자가 직접 상호작용하는 단계이고 다양한 프로그램 언어로 빌드된 앱을 포함
- GUI
- 사용자 인터페이스 제공 CUI 
- 시스템 리소스 사용 - 좀 더 간편하게 사용 할 수 있도록

#### 쉘
- 리눅스에서 명령어를 사용하는 창고, 명령어를 번역해서 사용하게되거나, 하드웨어를 이용
- 명령어를 해석하여 일련의 작동을 함
기능
- 명령어 해석
- 스크립트 실행: 쉘 스크립트를 통해 여러 명령어 조합이나 명령어를 일괄 실행
- 환경 관리: 환경 변수라 생각하면 됨
#### 커널
: 하드웨어 바로 위 단계, 운영체제의 주요 커널 단계
애플리케이션과 쉘은 커널을 통해 작동
기능
- 프로세스 관리:
- 메모리 관리
	- 가상 메모리: 보조기억 장치를 이용해 더 많은 메모리 사용
	- 페이징:  
	- 스왑 메모리: ? 이건 뭐 - 실제로 존재하는 파일처럼 보임
- 파일 시스템 관리: 
- 장치 관리
- 보안 및 접근 제어

#### 하드웨어
: 우리가 알고 있는 하드웨어 총합 


### Linux 종류
- 우분투
- centOS
- arch linux
- 알파인 리눅스 

#### 패키지 매니저
- APT
	- 버젼이 달라도 상위 버젼꺼의 패키지도 설치 가능
- YUM/DNF
- pacman
- zypper


#### Linux에서의 권한 및 소유자
- 모든 파일과 디렉토리는 사용자 User , 그룹 Group 에 의해 소유됩니다. 
	- 사용자 User : 파일을 생성한 주체로써 생성된 파일은 기본적으로 사용자에 의해 소유
	- 그룹 Group : 여러 사용자가 속해있는 그룹으로 여러사람이 같은 권한을 가질 수 있음
- 실습 명령어 - 한줄한줄 따라해보기

실행권한 1개의 파일은 3개의 권한을 가질 수 있음
- R(Read)/4 : 읽기 권한 - cat
- w(write)/2: 쓰기 권한 -  vim, echo, sed 
- x(eXecute)/1: 실행 권한 - sh, bash

- 한개의 파일이 갖는 권한은 User, Group, Others로 나뉩니다
	- ex. 755라면 User는 모든 권한 / Group은 읽기 실행 / Others 읽기 실행 
#### Linux root 디렉터리 구조
- root directory : 모든 파일과 프로세스는 이곳 하위에서 관리됨
 기본적으로 시스템 디렉토리라서 왠만하면 건들 ㄴㄴ
- /bin : 기본적인 사용자 명령어 저장
	- ls, cp, mv, rm, bash 등
- /lib : 공유 라이브러리들이 포함 
	- 심링크(심볼릭 링크 == 바로가기)
- /usr :  얘 밑에 bin, lib이 가지고 있다
- /dev: 장치 파일이 저장되는 디렉토리
	- 보통 안건들고 aws 쓸때 쓸듯
- /etc : 시스템 설정파일
	- 사용자 계정 정보, 암호 정보 ,등 저장하고 고 주로 텍스트 파일
- /opt: gui 를 쓸때 주로 씀 > 자주 안쓰일 예정
- /proc: 프로세스 
	- 커널과 프로세스 관련 파일이 저장되는 곳
	- /proc/cpuinfo(cpu 정보), /proc/meminfo(메모리 정보)등의 정보를 볼 수 있다.
- /home
- /root: 루트 계정의 홈 디렉토리 
	- 권한에 영향 잘 안받는 곳
#### 파일 시스템
- ext4 : 를 많이 씀
	- 대부분이 기본적으로 설치됨
#### 실습 명령어
0. 로그인 화면 - 아까 입력했던 아이디와 비밀번호 입력
1. service --status-all
	- ssh가 없을때 apt update && apt install -y openssh-server
	- 좀 더 쉽게 보는 법 service --status-all | grep ssh
2. service ssh start
	- 다시 서비스 명령어 쳐서 `[-]` 에서 `[+]`로 바꼈는지 확인하기
3. ip addr | grep inet
4. 터미널에서 직접 ssh로 접속하기
	- `ssh (사용자명)@(IP Address)`
	- ex.  ssh lyle@192.168.64.2
5. 명령어 실습 
	- sudo는 슈퍼권한? 
	- su 는 switching user 의 약자
	유저와 그룹 생성하는 실습
	- usermod(user modify) -aG(add Group) > 유저를 그룹에 넣는 명령어
		- 유저명:x: 권한: 그룹명
	- addgroup kakaotechbootcamp : 새로운 그룹 추가 
	- usermod -aG kakaotechbootcamp kakao : 카부캠에 카카오 사용자 추가
		- 사용자는 여러 그룹에 소속될 수 있고, 그룹 권한에 따라 여러 권한을 가질 수도 있다.
		- -aG: append 
	- cat /etc/group 으로 보이는 유저의 uid - 1000부터 시작
	 유저와 그룹이 가지는 권한 부여하기
	 - echo 'echo abcd' > test.sh
	 - ./test.sh > 권한 없음 뜸 
		 - 파일에 권한이 없기때문
		 - chmod +x test.sh 로 권한 부여 
		 - 권한이 1일때는 로컬에서는 커널 단위에서 이미 읽어들이고 실행하기때문에 실행권한만 있어도 되지만, 원격으로 접속하는 경우에는 읽어들이고 실행시키는 과정으로 인해 실행권한만 있다면 제대로 실행되지 않을 수도 있다. 
	- ls -al 파일이름 - 해당 파일의 부여된 권한 확인
	파일 소유자 조작
	- chmod 770 test.sh > other의 권한은 아무것도 가지지 않음
	- sudo chown kakao:kakao test.sh > 소유권을 기존 Lyle에서 kakao로 넘김
	 패키지 설치 해보기
	 - apt-get install nginx > 이걸 설치하면 어디에 설치될까? 를 보는 실습
	 - cd /etc/nginx - 여기에 실제로 사용되는 데이터가 있고 
	 - cd /usr/bin - 여기에 관련된 바이너리가 저장되어 있다
		 - bin 과 sbin의 차이점은 시스템에 있느냐 없느냐 차이
	 환경 변수 실행해보기
	 - cd /
	 - test.sh > 그냥 실행해보면 기존에 명령어가 있는지 찾고 없으니까 command not found 를 나타냄 
	 - export | grep PATH 
		 - PATH 의 경로를 순차적으로 돌며 test.sh이 있는지 쭉 찾아보는데 없다
	- export PATH=$PATH: /tmp
		- 이거하고 나면 /tmp 경로가 추가됨
	- sudo chown lyle:lyle test.sh - 테스트 파일 권한 가져오고
	- 다시 test.sh 실행
	- which test.sh : 현재 사용하고 있는 명령어의 파일 위치를 나타냄
	lsblk 명령어
	- 어떤 장치가 어떻게 얼마나 붙어 있는지 확인 할 수 있고
	- df 명령어를 통해서 os에 뭐가 붙었느지 확인 할 수 있다.
	- df -TH를 하면 데이터 단위까지 표시됨

- head /temp/abc : cat처럼 파일의 내용을 표시해주는 명령어
- tail /temp/abc: 얘도 cat처럼 파일의 내용을 표시해주는 명령어
- pwd: 현재 경로를 나타냄
- whoami: 현재 사용자가 누군지 나타냄

- top: 시스템 프로세스와 메모리 사용량 > 작업 관리자에서 사용량 어떤지 보는 개념인듯
- df -h : 파일 시스템의 디스크 공간 사용량을 데이터 단위로 표시
- du -h: 디렉토리와 파일의 디스크 사용량을 표시 -h로 데이터 단위로 표현됨
- curl:  터미널에서 데이터를 전송하거나 받아올 수 있는 도구
- ps: 현재 실행 중인 프로세스 목록 표시 > 작업 관리자 프로그램 탭 느낌
- ps -ef > 실행중인 프로세스 상세 정보 표시 -ef 옵션은 각 프로세스의 상세 정보를 표현

- ls -al /home/lyle | grep sh : `/home/lyle` 디렉토리 내의 모든 파일을 상세히 리스트하고, 그 중 'sh' 문자열이 포함된 파일이나 디렉토리만 출력합니다.
	- ls -al하면 파일 확장자명까지 다 표시되는데 여기서 sh 문자열이 포함한 모든 결과들을 출력함
- echo '{"key":"value"}' > test.json : test.json파일에 '{"key":"value"}'  를 입력 기존 파일이 있으면 덮어쓰기함
- cat test.json | jq  : test.json파일을 읽고 jq로 JSON 데이터를 보기 좋게 표현
	```json
	{
		"key": "value" // 이렇게 표현됨
	}
	```



알파인 리눅스
	
1. `git add something` -> Unversioned files가 Staging Area에 들어갑니다
    

NEW

3. _[_오후 4:29_]_
    
    `git stash` -> pull/merage

- 커밋 메세지를 잘 지키고 있는가? 
	- 팀내에서 어떠한 잘 지키고 있는가?
	- 보안 : 커밋 메세지에 보안 
- 배포

