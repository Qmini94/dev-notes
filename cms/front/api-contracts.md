# API 계약

이 문서는 프론트엔드와 백엔드 간의 **API 계약(API Contract)** 을 정의합니다.  
각 API는 요청(Request)과 응답(Response)의 구조, 인증 방식, 에러 규약을 포함하며,  
양측이 **동일한 기대값을 공유**하도록 하는 것을 목표로 합니다.

API 계약은 구현 세부 사항이 아니라  
**신뢰의 경계(boundary)** 로 취급됩니다.

---

## 기본 원칙

### 1. 계약 우선 (Contract First)

- API 구현 전에 계약이 먼저 정의되어야 합니다.
- 프론트엔드와 백엔드는 이 계약을 기준으로 독립적으로 개발할 수 있어야 합니다.
- 계약 변경은 구현 변경보다 우선적으로 문서에 반영되어야 합니다.

---

### 2. 일관된 응답 구조

모든 API 응답은 공통된 포맷을 따릅니다.

```json
{
  "success": true,
  "data": {},
  "message": "",
  "errorCode": null
}
- `success`: 요청 성공 여부
    
- `data`: 실제 응답 데이터
    
- `message`: 사용자 또는 디버깅용 메시지
    
- `errorCode`: 에러 식별 코드 (실패 시)
  ```
  
## 인증(Authentication) 관련 API

### 인증 방식

- 인증은 **토큰 기반(JWT)** 으로 처리됩니다.
    
- 인증 토큰은 HTTP Header를 통해 전달됩니다.
    

`Authorization: Bearer {accessToken}`

- 모든 보호된 API는 토큰 검증을 필수로 수행합니다.
    
- 토큰이 없거나 유효하지 않을 경우, 공통 에러 응답을 반환합니다.
    

---

### 사용자 정보 조회

#### `GET /api/auth/me`

- **설명**: 현재 로그인된 사용자 정보를 조회합니다.
    
- **인증**: 필요
    

**응답 예시**

`{   "success": true,   "data": {     "userId": "admin",     "userName": "관리자"   },   "message": "",   "errorCode": null }`

---

## 권한(Permission) 관련 API

### 권한 전파 방식

- 백엔드는 사용자 권한을 **명시적인 데이터 구조**로 응답합니다.
    
- 프론트엔드는 해당 정보를 기반으로 UI와 행동을 제어합니다.
    
- 프론트엔드에서 임의로 권한을 추론하지 않습니다.
    

---

### 메뉴/페이지 권한 조회

#### `GET /api/render/info`

- **설명**: 현재 페이지에 대한 렌더링 정보 및 권한을 조회합니다.
    
- **인증**: 필요
    

**응답 예시**

`{   "success": true,   "data": {     "siteId": "site001",     "menuId": "menu123",     "type": "board",     "permission": {       "access": true,       "view": true,       "write": false,       "modify": false     }   },   "message": "",   "errorCode": null }`

---

## 에러 처리 규약

### 공통 에러 응답

`{   "success": false,   "data": null,   "message": "권한이 없습니다.",   "errorCode": "AUTH_403" }`

### 에러 코드 정책

- 에러 코드는 문자열 상수로 관리합니다.
    
- 클라이언트는 에러 코드에 따라 분기 처리할 수 있습니다.
    
- 사용자 메시지와 시스템 에러 코드는 분리합니다.
    

---

## API 변경 정책

- 기존 필드 삭제 ❌
    
- 필드 추가 ⭕ (하위 호환성 유지 시)
    
- 필드 의미 변경 ❌
    

계약 변경 시 반드시 다음을 수행해야 합니다.

- 문서 업데이트
    
- 변경 이력 기록
    
- 프론트/백엔드 간 사전 공유
    

---

## 문서 연계

이 문서는 다음 문서들과 함께 사용됩니다.

- data-models.md
    
- architecture.md
    
- development-guide.md
    

API 계약은 구현보다 우선하며,  
모든 구현은 본 문서를 기준으로 검증됩니다.