/*
[FRONT Jenkinsfile 요구사항]

0) 공통 정책
- Pipeline as Code: Jenkinsfile은 Git 저장소 루트에서 관리
- 동시 실행 방지(disableConcurrentBuilds)
- 로그 타임스탬프/가독성(ansiColor, timestamps)
- 실패 시 원인 파악 가능하도록 단계별 echo/log 출력

1) Checkout
- Gitea repo에서 지정 브랜치(main) 체크아웃
- Jenkins credentials(GIT_CREDENTIALS_ID) 사용

2) 환경 주입(.env)
- 운영 환경변수는 Jenkins Credentials(Secret file)로 관리
- 빌드 전에 .env로 복사하고, 배포 전/후에는 workspace에 남기지 않도록 삭제(cleanup)

3) Build 준비 (운영 데이터 public 보존)
- 운영 서버에만 존재하는 public/ (업로드/정적 데이터)가 있을 수 있음
- 배포 시 rsync --delete로 인해 운영 public이 삭제되지 않도록,
  빌드 전에 운영 서버의 public/을 rsync로 workspace에 pre-sync
- Nuxt build 시 workspace/public → .output/public로 복사되므로
  pre-sync는 빌드 산출물에 운영 public을 포함시키는 목적

4) Install & Build
- nvm 기반 Node 버전 고정(NODE_VERSION=22)
- npm ci (package-lock 존재 시) / 없으면 npm install
- npm run build 수행 (Nuxt)

5) Artifact 보관
- build.tar.gz 생성(불필요 폴더 제외: node_modules/.git/.nuxt/.output 등)
- archiveArtifacts로 Jenkins에 저장(fingerprint)
- 목적: 배포 실패/롤백/이력 관리

6) Deploy
- DRY_RUN=true일 때 배포 단계는 스킵 (빌드/아카이브만 수행)
- 배포 단계는 수동 승인(input) 옵션 제공(DEPLOY_CONFIRM)
- rsync로 운영 서버 DEPLOY_DIR로 반영
- 필요 시 서버에서 npm install 수행(운영 의존성 보장)
- systemd로 nuxt 재시작(systemctl restart nuxt)

7) 안정성/보안
- rsync --delete 사용 시 운영 데이터 손실 위험 → pre-sync/보호 전략 필요
- ssh/rsync/scp는 지정 포트(SSH_PORT) 사용
- .env 등 민감 파일이 아카이브/서버에 남지 않도록 방지

[BACKEND Jenkinsfile 요구사항]

0) 공통 정책
- Pipeline as Code: Jenkinsfile은 Git 저장소 루트에서 관리
- 동시 실행 방지(disableConcurrentBuilds)
- 로그 타임스탬프/가독성(ansiColor, timestamps)
- 실패 시 원인 파악 가능하도록 단계별 echo/log 출력

1) Checkout
- Gitea repo에서 지정 브랜치(main) 체크아웃
- Jenkins credentials(GIT_CREDENTIALS_ID) 사용
- (선택) Multibranch 사용 시 BRANCH_NAME 기반으로 동작 가능

2) Build (Gradle)
- gradlew 실행 권한 보장(chmod +x gradlew)
- ./gradlew clean build -x test (초기에는 테스트 제외, 추후 CI 강화 시 test 포함)
- 빌드 산출물 jar 생성 확인(build/libs/*.jar)

3) 산출물 버저닝/식별
- jar 파일을 빌드 번호 기반으로 리네이밍(app-${BUILD_NUMBER}.jar)
- (권장) 커밋 해시(short SHA)도 함께 남기면 롤백/추적 용이

4) Artifact 보관
- archiveArtifacts로 jar 저장(fingerprint)
- 목적: 같은 빌드 산출물로 재배포/롤백 가능하게 만들기

5) Deploy 트리거
- DRY_RUN=true일 때 배포 단계는 스킵 (빌드/아카이브만 수행)
- 배포 단계는 수동 승인(input) 옵션 제공(DEPLOY_CONFIRM)

6) Deploy 방식(원격)
- scp로 jar를 운영 서버 DEPLOY_DIR로 업로드
- 운영 서버에서 deploy.sh 실행하여 실제 배포 처리
  (교체/재시작/헬스체크/롤백/무중단 전략은 deploy.sh에서 담당하는 구조 권장)

7) 무중단 배포(향후 확장 요구사항)
- Blue/Green 또는 포트 스위칭 방식 고려
- deploy.sh에서 다음 로직 제공 필요:
  - 새 인스턴스 기동(다른 포트/JVM)
  - health check 통과 확인
  - 트래픽 전환(Nginx upstream 변경 등)
  - old 인스턴스 종료
- 자원 제약(현재 vCPU 1 등) 고려하여 동시 2프로세스 실행 가능 여부 점검 필요

8) 안정성/운영
- 배포 실패 시 이전 artifact로 재배포(rollback) 가능해야 함
- 배포/재시작/헬스체크 로그를 서버 측에 남겨 추적 가능하도록 구성 권장
- SSH/포트/권한/경로는 환경변수로 통일 관리
*/
