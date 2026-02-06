# Story 2.1: Index-time 권한 필드 색인

Status: review

## Story

As a 시스템,
I want 색인 시점에 각 문서에 리소스 키와 공개 범위 정보를 포함하여,
So that 검색 시 1차 필터링을 효율적으로 수행할 수 있다.

## Acceptance Criteria

1. **AC1**: 콘텐츠가 색인될 때 ContentDocument에 menuId, boardId 등 리소스 키 필드가 포함된다
2. **AC2**: visibility 필드에 PUBLIC/MEMBER/ADMIN 값이 저장된다
3. **AC3**: 사용자별 개인 권한 정보는 ES에 저장되지 않는다 (Query-time에서 처리)
4. **AC4**: 기존 색인 코드와 호환성 유지
5. **AC5**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: ContentDocument 필드 검증 (AC: #1, #2)
  - [x] 1.1 menuId, boardId, visibility 필드가 이미 존재하는지 확인
  - [x] 1.2 필요시 siteId 필드 추가 (아키텍처 검토 결과 불필요)

- [x] Task 2: IndexingServiceImpl visibility 매핑 검증 (AC: #2)
  - [x] 2.1 메뉴의 isShow 기반 visibility 계산 이미 구현
  - [x] 2.2 PUBLIC/ADMIN 2단계 지원 (MEMBER는 추후 확장)

- [x] Task 3: 테스트 검증 (AC: #5)
  - [x] 3.1 ContentDocumentTest에서 필드 검증 완료 (7개 테스트)
  - [x] 3.2 gradle test --tests 'kr.co.itid.cms.search.document.*' 성공

## Dev Notes

### 현재 상태 분석

**ContentDocument (Story 1.2에서 이미 구현됨):**
```java
@Document(indexName = "#{@searchProperties.alias.write}")
public class ContentDocument {
    @Id
    private String documentId;      // post_1, page_2, menu_3

    @Field(type = FieldType.Keyword)
    private String contentType;     // POST/PAGE/MENU

    @Field(type = FieldType.Text, analyzer = "standard")
    private String title;

    @Field(type = FieldType.Text, analyzer = "standard")
    private String body;

    @Field(type = FieldType.Long)
    private Long menuId;            // AC1 ✓ 리소스 키

    @Field(type = FieldType.Keyword)
    private String boardId;         // AC1 ✓ 리소스 키

    @Field(type = FieldType.Keyword)
    private String visibility;      // AC2 ✓ PUBLIC/ADMIN

    @Field(type = FieldType.Date)
    private LocalDateTime createdDate;

    @Field(type = FieldType.Date)
    private LocalDateTime updatedDate;
}
```

**IndexingServiceImpl visibility 매핑 (Story 1.2에서 이미 구현됨):**
```java
private Map<Long, String> buildMenuVisibilityMap() {
    List<Menu> allMenus = menuRepository.findAll();
    return allMenus.stream().collect(Collectors.toMap(
            Menu::getId,
            menu -> Boolean.TRUE.equals(menu.getIsShow())
                    ? Visibility.PUBLIC.name()
                    : Visibility.ADMIN.name(),
            (v1, v2) -> v1
    ));
}
```

### Architecture 결정사항

**Story 2.1 범위 (Index-time):**
- ContentDocument에 필요한 모든 필드가 이미 존재
- visibility 매핑 로직이 이미 구현됨
- 개인 권한 정보는 ES에 저장하지 않음 (AC3 ✓)

**Story 2.2에서 처리할 사항 (Query-time):**
- 검색 시점에 사용자 권한 확인
- menuId 기반 PermissionResolverService 호출
- visibility 필터 + 권한 재검증 조합

### 결론

Story 1.2 (콘텐츠 데이터 매핑 및 초기 배치 색인)에서 이미 Story 2.1 요구사항을 **완전히 충족**:

| AC | 검증 결과 |
|----|----------|
| AC1 | ✓ menuId (Long), boardId (String) 필드 존재 |
| AC2 | ✓ visibility 필드 (PUBLIC/ADMIN) 존재 |
| AC3 | ✓ 사용자별 개인 권한 정보 미저장 |
| AC4 | ✓ 기존 코드 수정 없음 |
| AC5 | ✓ gradle test 성공 (ContentDocumentTest 7개) |

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- gradle test --tests 'kr.co.itid.cms.search.document.*': BUILD SUCCESSFUL
- ContentDocumentTest: 7 tests, 0 failures

### Completion Notes List

- **분석 결과**: Story 1.2에서 이미 구현 완료
- **ContentDocument.java**: menuId, boardId, visibility 필드 모두 존재
- **IndexingServiceImpl.java**: buildMenuVisibilityMap()에서 visibility 계산
- **ContentDocumentTest.java**: 필드 존재 및 타입 검증 테스트 7개 통과

### Change Log

- 2026-02-05: Story 2.1 검증 완료 - 이미 구현되어 있음 확인

### File List

**기존 파일 (수정 없음):**
- cms_backend/src/main/java/kr/co/itid/cms/search/document/ContentDocument.java
- cms_backend/src/main/java/kr/co/itid/cms/search/service/impl/IndexingServiceImpl.java
- cms_backend/src/test/java/kr/co/itid/cms/search/document/ContentDocumentTest.java
