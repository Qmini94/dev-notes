# 박규민

`+821025780750` `pkm1614@gmail.com`

4년 차 Spring Boot 기반 백엔드 엔지니어로서 CI/CD 무중단 배포 파이프라인, Redis 기반 인증/캐싱, Elasticsearch 통합검색을 설계/구축 경험을 보유하고 있습니다. 기존 솔루션을 도메인 분리와 확장 가능한 구조로 마이그레이션하며 구조 개선과 운영 효율성을 함께 고려한 설계를 수행해왔습니다.

### 업무를 대하는 철학과 생각

신입 시절, 저는 개발을 단순히 기능을 구현하는 것으로 정의했습니다. 하지만 레거시 시스템을 마이그레이션하고 운영 이슈를 해결하는 과정에서 개발을 "안정적이고 지속 가능한 시스템을 만드는 것"으로 정의하게 되었습니다.

예기치 못한 장애들과, 복잡한 코드로 인해 유지보수에 많은 비용을 사용했던 경험은 저에게 값진 교훈이 되었습니다. 이를 통해 방어적인 개발, 클린 코드, 체계적인 형상 관리, 문서 작업이 개인의 만족을 넘어 팀 전체의 생산성과 직결된다는 것을 체득했습니다.

저에게 경험이란 성장의 가장 큰 자산입니다. 하지만 혼자만의 경험에 갇히지 않고, 동료들과의 적극적인 기술 공유와 소통을 통해 간접 경험까지 흡수하려 노력합니다. 경험을 통해 단단해진 노하우와 열린 태도로, 팀의 기술적 난제를 함께 해결하고 안정적인 서비스를 만들어가는 든든한 동료가 되고 싶습니다.

### 개발자로서 목표와 비전

비즈니스의 성장을 기술로 뒷받침하는 엔지니어로서, 대규모 트래픽 처리가 가능한 아키텍처를 설계하고 운영하는 환경에서 제 역량을 발휘하고 싶습니다. 현재 AI 영향과 빠르게 변화하는 기술 트렌드 속에서도 변하지 않는 가치는 시스템의 안정성이라고 생각합니다. 이를 바탕으로 조직에 기여하고자 합니다.

다양한 운영 이슈나 개발을 팀원들과 함께 헤쳐나가며 탄탄한 기본기를 다졌습니다. 이러한 경험을 발판 삼아, 주어진 문제를 기술적으로 해결하는 것을 넘어, 비즈니스 목표를 팀원들과 함께 고민하고 실천하는 일원이 되겠습니다. 동료들에게는 신뢰를, 서비스에는 안정성을 더하는 엔지니어로 조직과 함께 성장하는 미래를 그려가고 싶습니다.

---

## 경력 3년 5개월

### (주)아이디

`2024.01 ~ 2026.06 (2년 6개월)` | 정규직 | Backend Developer | 연구원

---

**유지보수 관리 시스템 — 설계/개발**
`2026.01 ~ 현재`

`Next.js 15 / React 19 / Node.js / TypeScript / Prisma / MySQL / Redis / NextAuth.js / TanStack Query / Tailwind / shadcn/ui / Playwright / Docker`

> 관공서 60여 곳 포함 **300여 고객사**의 유지보수 서비스를 통합 관리하는 B2B 웹 플랫폼. 레거시 전면 대체, 1인 설계·개발.

- 관공서/교육기관/기업 대상, 접수 → 할당/재할당/협업 → 상태 추적 → 완료까지 단일 시스템으로 완결
- Next.js 15 App Router + React 19 Server/Client Components 기반 풀스택 설계
- NextAuth.js v5 + Middleware 4역할 RBAC, JWT + Redis 세션
- CQRS-유사 구조 + 도메인 이벤트(EventEmitter)로 인앱/Slack 알림 분리
- 월간 보고서 PDF/Excel 자동 생성, 역할별 3종 대시보드
- AES-256 민감정보 암호화, 소프트 삭제, Vitest/Playwright 테스트
- Claude Code + BMad 기반 AI 개발 워크플로로 설계·구현

---

**Q-CMS 솔루션 개발**
`2025.04 ~ 2026.01`

`Spring Boot / JPA / QueryDSL / eGovFrame / Nuxt 3 / Vue / jQuery / Elasticsearch / Redis / MySQL / Jenkins / Docker`

> 레거시 CMS를 모던 스택으로 리빌딩·마이그레이션. 검색·인증·배포·게시판·UI 전반 설계.

- Elasticsearch + Nori 기반 80만 건 통합검색 (무중단 재색인, 색인 일관성 보장)
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

`2022.03 ~ 2023.01 (11개월)` | 정규직 | Backend Developer | 연구원

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

## 학력

**한양사이버대학교** · 컴퓨터IT공학과 `2025.03 ~ 2027.02 (졸업 예정)`

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

## 자격 / 교육

- BITCAMP Framework 전문 개발자 양성과정 `2021.07`
- 넷츠(NETS) 보안 솔루션 내부 교육 수료 `2022`
