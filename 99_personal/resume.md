# 박규민

`+821025780750` `pkm1614@gmail.com`

4년 차 풀스택 엔지니어로서 Next.js/React 기반 B2B 웹 플랫폼을 1인 설계/개발한 경험과, Spring Boot 기반 CMS 솔루션의 리빌딩/운영 경험을 보유하고 있습니다. Elasticsearch 통합검색, CI/CD 무중단 배포, Redis 기반 인증/캐싱을 직접 설계/구축하였으며, 레거시 시스템의 구조 개선과 서비스 안정화를 함께 수행해왔습니다.

### 업무를 대하는 철학과 생각

신입 시절, 저는 개발을 단순히 기능을 구현하는 것으로 정의했습니다. 하지만 레거시 시스템을 마이그레이션하고 운영 이슈를 해결하는 과정에서 개발을 "안정적이고 지속 가능한 시스템을 만드는 것"으로 정의하게 되었습니다.

예기치 못한 장애들과, 복잡한 코드로 인해 유지보수에 많은 비용을 사용했던 경험은 저에게 값진 교훈이 되었습니다. 이를 통해 방어적인 개발, 클린 코드, 체계적인 형상 관리, 문서 작업이 개인의 만족을 넘어 팀 전체의 생산성과 직결된다는 것을 체득했습니다.

저에게 경험이란 성장의 가장 큰 자산입니다. 하지만 혼자만의 경험에 갇히지 않고, 동료들과의 적극적인 기술 공유와 소통을 통해 간접 경험까지 흡수하려 노력합니다. 경험을 통해 단단해진 노하우와 열린 태도로, 팀의 기술적 난제를 함께 해결하고 안정적인 서비스를 만들어가는 든든한 동료가 되고 싶습니다.

### 개발자로서 목표와 비전

실제 고객이 사용하는 서비스를 안정적으로 운영하면서 꾸준히 개선해나가는 일에 가장 보람을 느낍니다. B2B 웹 플랫폼을 1인으로 설계부터 배포까지 담당한 경험을 통해, 서비스의 전체 흐름을 파악하고 필요한 부분을 스스로 판단하여 실행하는 습관을 갖추게 되었습니다.

다양한 운영 이슈를 팀원들과 함께 헤쳐나가며 탄탄한 기본기를 다졌습니다. 이러한 경험을 발판 삼아, 주어진 문제를 기술적으로 해결하는 것을 넘어, 제품의 방향성을 함께 고민하고 더 나은 구조를 제안하며 실천하는 일원이 되겠습니다. 동료들에게는 신뢰를, 서비스에는 안정성을 더하는 엔지니어로 조직과 함께 성장하는 미래를 그려가고 싶습니다.

---

## 경력 3년 5개월

### 네이버 SEO 분석 솔루션 · 개인 프로젝트 (진행 중)

`Next.js 16 / React 19 / TypeScript / Node.js / Prisma / PostgreSQL / Redis / BullMQ / NextAuth.js / Puppeteer / Recharts`

> 시드 키워드로 네이버 검색 생태계에서 '이길 수 있는 기회 키워드'를 찾아주고, 키워드별 **콘텐츠 처방전**(채널·구조·AI 브리핑 최적화)을 생성하는 B2B SEO 분석 도구. **기획·설계·개발 단독 진행.**

- 네이버 검색광고 API·DataLab·검색 API 연동으로 검색량/경쟁도 등 비공개 수요 데이터 수집
- 수요, 경쟁, 관련성 기반 **기회 점수 산출** + 검색 의도 분류로 우선순위 키워드 선별
- Puppeteer 기반 **SERP 크롤링** — 영역별 노출 구조와 상위 콘텐츠 구조(글 길이·키워드 밀도·소제목·이미지)를 분석해 콘텐츠 처방전 생성
- 네이버 **AI 브리핑 인용 패턴 분석** — 신규 영역 화이트스페이스 공략 설계
- **레이어드 아키텍처**(Route Handler → Service → Repository → Infra) + 도메인 모듈화, zod 검증, 인터페이스 추상화 기반 테스트 설계
- BullMQ 워커로 크롤링 비동기 처리, Redis 캐싱으로 외부 API 호출 제한 대응
- 현재: 검색광고 API 연동·스키마·키워드 리서치 구현 중 / SERP 분석·처방전 단계 설계 완료

### (주)아이디

`2024.01 ~ 2026.06 (2년 6개월)` | 정규직 | Full-Stack Developer | 연구원

---

**유지보수 관리 시스템 — 설계/개발**
`2026.01 ~ 현재`

`Next.js 15 / React 19 / Node.js / TypeScript / Prisma / MySQL / Redis / NextAuth.js / TanStack Query / Tailwind / shadcn/ui / Playwright / Docker`

> 관공서 60여 곳 포함 **300여 고객사**의 유지보수 서비스를 통합 관리하는 B2B SaaS 웹 플랫폼. 레거시 전면 대체, 1인 설계·개발·운영.

- 접수 → 할당/재할당/협업 → 상태 추적 → 완료까지 고객사 유지보수 워크플로 단일 시스템으로 완결
- Next.js 15 App Router + React 19 Server/Client Components 기반 풀스택 설계
- NextAuth.js v5 + Middleware 4역할 RBAC, JWT + Redis 세션
- CQRS-유사 구조 + 도메인 이벤트(EventEmitter)로 인앱/Slack 알림 분리
- 월간 보고서 PDF/Excel 자동 생성, 역할별 3종 대시보드 — 고객/운영팀 피드백 기반 반복 개선
- 외부 API 연동 및 예외 처리, 데이터 흐름 모니터링
- AES-256 민감정보 암호화, 소프트 삭제, Vitest/Playwright E2E 테스트
- 배포·장애 대응·운영 매뉴얼 문서화까지 서비스 전체 사이클 담당

---

**Q-CMS 솔루션 개발**
`2025.04 ~ 2026.01`

`Spring Boot / JPA / QueryDSL / eGovFrame / Nuxt 3 / Vue / jQuery / Elasticsearch / Redis / MySQL / Jenkins / Docker`

> 레거시 CMS를 모던 스택으로 리빌딩·마이그레이션. 검색·인증·배포·게시판·UI 전반 설계. 관리자/백오피스 기능 전담.

- Elasticsearch + Nori 기반 80만 건 통합검색 — 한국어 형태소 분석, 권한 필터 포함 검색, 무중단 재색인
- 런타임 DDL 기반 동적 게시판 엔진 — 관리자가 게시판 구조를 직접 생성·운영
- 콘텐츠–메뉴 자동 매핑 API 설계
- JWT + Redis 인증, Redis 캐싱·pathId 구조로 권한 조회 성능 개선
- Redis Streams 기반 비동기 감사 로깅 (AOP)
- Jenkins Blue/Green 무중단 배포 (BE/FE 독립 파이프라인)
- Nuxt 3 SSR/CSR 하이브리드 렌더링 — 미들웨어 5단 파이프라인(IP 제어·인증·메뉴·권한·레이아웃) 분리 설계, Static JSON + Spring Cache 2단계 캐싱으로 Lighthouse 93점
- 게시판 유형별 테마 컴포넌트 동적 조합 렌더링 — 리스트·검색·페이지네이션 슬롯 구성, 신규 테마 추가 시 라우터·빌드 변경 없이 디렉토리 단위 확장
- jQuery → Vue 3 Composition API 점진적 마이그레이션 — 레거시 페이지 단위 전환, Pinia 상태 관리 도입
- 클라이언트 세션 동기화 — 서버 만료 시각 기반 자동 로그아웃 타이머, 토큰 갱신 인터셉터 처리
- CodeMirror 6 기반 실시간 CSS 에디터, 드래그/드롭 트리 메뉴 관리 UI, 다크모드 전환

---

**강진군청 홈페이지 전면개편 — 기술 리드**
`2024.10 ~ 2025.03`

`PHP / Python / jQuery / MySQL / Apache`

> 강진군청 공식 홈페이지 전면개편 프로젝트. 설계부터 오픈까지 기술 리드로 참여.

- 기존 홈페이지 구조 분석 및 신규 IA(정보구조) 설계
- 페이지 템플릿·공통 컴포넌트 설계, 프론트/백엔드 개발 리드
- 관공서 웹 접근성·호환성 기준 충족, 검수 대응

---

**행정전화번호부 — 6개 시·군청 구축·배포**
`2024.06 ~ 2024.09`

`Python / Java / JavaScript / MySQL / Apache / ETL`

> 전남·경북·울산 등 6개 지자체 행정전화번호부 웹앱 구축 및 정부 폐쇄망 데이터 연계.

- 6개 시·군청 대상 행정전화번호부 시스템 구축·배포
- 정부 폐쇄망 API 기반 조직·인사 데이터 ETL 연계 파이프라인 구축
- 지자체별 조직 구조·데이터 포맷 차이에 대한 정제·변환 로직 설계

---

**재정 데이터 이기종 DB 마이그레이션**
`2024.03 ~ 2024.05`

`Linux / Python / Java / MySQL / Apache / ETL`

> 재정·계약 등 핵심 데이터의 Oracle → MySQL 이기종 마이그레이션 및 레거시 스키마 재설계.

- 레거시 Oracle 스키마 분석, MySQL 기준 테이블·관계 재설계
- 복합 조인 쿼리 이기종 변환 및 데이터 정합성 검증
- 대량 데이터 Excel 출력 처리 — 대용량 시트 분할, 메모리 최적화

---

**개발 문화 정착 & 사내 교육**
`2024.01 ~ 현재`

> Git 협업·CI/CD·문서화 문화를 직접 학습/도입하여 사내에 정착시킴.

- Git 브랜치 · PR 기반 협업 워크플로 도입 및 사내 정착
- Jenkins CI/CD 무중단 배포 파이프라인 구축·표준화
- 사내 개발 교육 진행 및 가이드 문서 제작·배포

---

### 낸드소프트

`2022.03 ~ 2023.01 (11개월)` | 정규직 | Full-Stack Developer | 연구원

---

**보안 솔루션 기반 개발 및 커스터마이징**
`2022.03 ~ 2022.09`

`넷츠(NETS) / Spring Boot / SAML / LDAP·AD`

> 넷츠(NETS) 보안 솔루션 커스터마이징. 대구은행·부산은행·태광기업 등 금융·대기업 클라이언트 대상.

- 클라이언트별 보안 솔루션 기능 개발·커스터마이징
- SAML 2.0 기반 SSO 통합 인증 연동
- Active Directory(LDAP) 연동 — 인증 및 사용자·그룹·권한 동기화
- 레거시 Spring → Spring Boot 마이그레이션, 조직도 트리 UI 설계

---

**넷츠 본사 보안 솔루션 내부 교육 수료**
`2022.09 ~ 2022.12`

> 넷츠 본사(서울) 파견, 보안 솔루션 아키텍처 및 내부 동작 구조 교육 수료.

---

## 스킬

**Languages** Java / TypeScript / JavaScript / Node.js / PHP
**Backend** Spring Boot / JPA / QueryDSL / eGovFrame / Next.js / Node.js / Prisma
**Database** MySQL / Oracle
**Cache / Search** Redis / Elasticsearch
**Frontend** React / Next.js 15 / Vue / Nuxt 3 / jQuery / Pinia / TailwindCSS / shadcn/ui / CodeMirror
**Infra / DevOps** Jenkins(Blue/Green) / Docker / Linux / Apache / Nginx / Git / Gitea / k6 / ETL
**Test** Vitest / Playwright / RTL / MSW
**Auth** SAML SSO / Active Directory(LDAP) / JWT / NextAuth.js
**Docs / 협업** Git PR 워크플로 / Markdown·Obsidian 문서화 / Slack
**기타** Claude Code / BMad

## 학력 / 자격

- **한양사이버대학교** · 컴퓨터IT공학과 `2025.03 ~ 2027.02 (졸업 예정)`
- BITCAMP Framework 전문 개발자 양성과정 `2021.07`
- 넷츠(NETS) 보안 솔루션 내부 교육 수료 `2022`
