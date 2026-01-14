# 데이터 모델

이 문서는 프로젝트에서 사용되는 **핵심 데이터 모델과 타입 정의**에 대한 개요를 제공합니다.  
각 모델은 애플리케이션의 특정 도메인(인증, 사이트, 렌더링 등)을 중심으로 설계되었으며,  
프론트엔드 전반에서 **일관된 데이터 구조와 의미**를 유지하는 것을 목표로 합니다.

구체적인 타입 정의는 도메인별로 정리된  
`/types` 디렉토리에서 확인할 수 있습니다.

---

## 핵심 인증 및 사용자 모델

이 모델들은 애플리케이션의 **인증(Authentication)** 및 **권한(Permission)** 시스템의 중심을 이룹니다.

### `UserInfo`

- **위치**: `types/user/info.ts` (추정)
- **스토어**: `userStore`
- **설명**: 현재 로그인된 사용자를 표현하는 모델입니다.
- **주요 필드**:
    - `userId: string`  
      사용자의 로그인 ID
    - `userName: string`  
      사용자 표시 이름

---

## 사이트 및 렌더링 관련 모델

이 모델들은 **사이트 구조**, **메뉴 렌더링**, **페이지 권한 처리**와 직접적으로 연관됩니다.

### `RenderInfo`

- **위치**: `types/site/render.ts` (추정)
- **스토어**: `renderStore`
- **설명**: 현재 렌더링 중인 페이지에 대한 메타 정보를 포함합니다.
- **주요 필드**:
    - `siteId: string`  
      현재 사이트 식별자
    - `menuId: string`  
      현재 메뉴 식별자
    - `type: string`  
      렌더링할 페이지 유형 (예: `"board"`, `"content"`)
    - `option: BoardOption | any`  
      페이지별 옵션 정보 (게시판 설정 등)
    - `permission: BoardPermission`  
      현재 사용자에게 적용되는 페이지 권한 정보

### `BoardPermission`

- **위치**: `types/site/render.ts` (추정)
- **스토어**: `renderStore`
- **설명**: 사용자가 현재 페이지에서 수행할 수 있는 행위를 정의합니다.
- **주요 필드**:
    - `access: boolean`  
      페이지 접근 가능 여부
    - `view: boolean`  
      콘텐츠 조회 가능 여부
    - `write: boolean`  
      신규 콘텐츠 작성 가능 여부
    - `modify: boolean`  
      기존 콘텐츠 수정 가능 여부
