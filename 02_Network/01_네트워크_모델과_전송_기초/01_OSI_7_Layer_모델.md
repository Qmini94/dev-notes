## 목적
- 네트워크 문제를 계층 단위로 분리하기 위한 개념 모델
- 실제 구현이 아니라 문제 원인 분석을 위한 좌표계

## 계층 요약
1. Physical
2. Data Link
3. Network
4. Transport
5. Session
6. Presentation
7. Application

## 실무 관점 정리
- ping은 되는데 HTTP가 안 된다 → L3 OK, L7 문제
- 포트가 막혔다 → L4 문제
- 인증/쿠키 문제 → L7 문제

## 정리
- 외우는 용도 아님
- 문제 범위 좁히는 용도로 사용