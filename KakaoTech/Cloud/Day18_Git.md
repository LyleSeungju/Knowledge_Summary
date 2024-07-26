#### 소스 코드 관리?
- 백업 
- 버전 관리
- 협업 관리
- 보안 관리

#### 어떤 도구를 사용하여 소스코드를 관리 할 수 있을까?
- SCM software configuration Management  형상 관리 도구 == 버젼 관리 도구
	- CVS : 거의 안씀 ~2000
	- SVN : ~2010
		- 폴더 중심으로 checkout / commit / push 진행
		- 서버 중앙에서 push에 따른 conflict 제어 > 사람이 많을수록 피곤
		- 모두 하나의 브랜치(폴더)에서 수정된 파일을 서버에 직접 업로드
		- 단순하고 직관적이고 commit/push 를 가능한 신중히
	- GIT : 2010~
		- 브랜치 중심 checkout/commit/push 진행
		- 로컬에서 직접 관리 
		- 개발자는 로컬의 별도의 브랜치를 갖고 개발하여, local history 관리가 가능하고 개발된 사항을 별도로 업로드
		- 복잡하지만, commit/push 브랜치 기준으로 하기 때문에 다수의 개발자들과 협업하기에 용이

#### Git 브랜치 전략
: 여러사람의 각자의 브랜치? >> 각자의 태스크
- 어떤 기준으로 나누나? **소프트웨어 개발 정책**에 따라
- 소프트웨어 개발 정책 
	- 개발 Develop  > 검증 QA > 배포 Production
	- 정적 소스코드 검사 Lint나 코드 리뷰 등을 어떤 브랜치에서 할지 고민
	- 테스트코드를 어떤 브랜치에 적용할지 고민 > 자주사용하는 패턴이 있다
- 제품 관리 정책
	- 고객의 요구사항에 따라 하나의 제품이 여러 버전으로 구별 될 수 있음
	- OSMU(one-source multi-use)를 유지할 것이냐, 제품을 나눌 것이냐

기본 브랜치
- master : qa가 끝난 최신 버전의 브랜치
- develop: 최신 버젼은 맞는데 qa까지는 아닌
개발용 브랜치
- feature/{업무id or 이름} : 기능 기준으로 feature을 추가하기 위해
- hotfix/{업무id or 이름 or 날짜} : 빠르게 배포하기 위한것
배포용 브랜치
- release/{날짜} : 배포하기 전 qa를 진행하기 위한 브랜치

#### Git-Flow 도구 활용
- 브랜치 관리 방식을 git-flow가 잡아줌 
- `git flow hotfix finish -Fp {브랜치명} `
- 구름에서 Clickup 도구?

#### Git Repository 웹 서비스
- 보통 Github / Gitlab, Bitbucket

#### 정적 소스코드 검사
- 커밋 시 소스코드를 검사하여 컨벤션 체크, 문법 체크
- git hooks
- husky
- 프론트엔드에서는 husky-> git hooks 설정한뒤 lint-staged 이용해서 ESLint 사용
#### 코드 리뷰
- 보통 feature 브랜치를 develop 브랜치 머지 전에 코드 리뷰
#### 테스트 코드 적용
- 커밋 전 유닛 테스트
- 필요에 따라 CI/CD에 테스트 코드를 넣고 실패하면 배포하지 않는 경우도 있음
- Jenkins, argoCD 등에 E2E Test 많이 적용 / E2E Test 는 playwright 많이 사용

#### 코드 리뷰를 위한 Pull Request (PR) 전략

메인 라인 브랜치(develop) 머지 프로세스
1. 개발자가 목적에 맞게(feat) 코드를 생성, 수정 진행 후 푸시

#### Merge 전략
1. Merge 
	- 누가 언제 어떤 커밋을 한지 볼때
1. Squash and Merge 
	- 커밋이 엄청 많을때 > 나중에 수정사항 찾기가 어려움
2. Rebase and Merge
	- 브랜치가 안 나뉘도 커밋 별로 관리 > 커밋 하나하나가 중요할때

#### 제품 관리 정책 기준 GIT 브랜치 전략
기능을 옵션화 vs 별도의 제품 구별
- 기능 옵션화를 하면 OSMU 가능 > 기능이 복잡해짐
- Configuration?

#### Git Tag 전략
- X.Y.Z 버전을 통한 업데이트 명시
	- X: major 업데이트. 새로운 기능이 추가되었을때나 전체적인 업데이트 발생
	- Y: minor 업데이트. 큰 기능에서 조금의 기능이 업데이트 되거나 개선되었을때
	- Z: fix 업데이트. 오류 수정 및 성능 개선 되었을 때

#### 좋은 정책 또는 컨벤션
- 기본 브랜치 정책
- feature, hotfix에 대한 정책
- git commit 컨벤션
- PR 정책

## 실습 2
- 다른 브랜치에 있는 특정 커밋을 내 브랜치로 옮기기
- 환경 - main branch / feat branch
1. main 브랜치에 커밋 2개정도 남겨두기
2. `git pull` : 리모트(깃헙)에 있는 모든 데이터 업데이트
3. `git log origin/main` : 여기서 최근 커밋 한개의 해시를 복사해두기
5. `git branch -a` : 브랜치 리스트 보기(만약 내가 만든 브랜치가 없다면 `git fetch` 다시 시도)
6. `git checkout <바꾸려는 브랜치명>` : 해당 브랜치로 설정
7. `git cherry-pick <아까 복사한 해시>` 
8. `git push` : 하면 메인 브랜치에 있던 특정 커밋을 내가 원하는 브랜치로 들고 올 수 있다. 
