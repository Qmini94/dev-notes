# Story 4.2: 기존 CMS Service에 이벤트 발행 통합

Status: review

## Story

As a 콘텐츠 작성자,
I want 게시글/페이지/메뉴를 생성·수정·삭제하면 검색 인덱스에 자동 반영되어,
So that 변경된 콘텐츠가 5분 이내에 검색 가능하다.

## Acceptance Criteria

1. **AC1**: 게시글을 생성할 때 BoardServiceImpl.create가 실행되면 IndexingEvent(CREATE, POST, idx)가 발행되고 5분 이내 검색 결과에 반영된다
2. **AC2**: 페이지 콘텐츠를 수정할 때 ContentServiceImpl.update가 실행되면 IndexingEvent(UPDATE, PAGE, idx)가 발행되고 변경 내용이 검색에 반영된다
3. **AC3**: 메뉴를 삭제할 때 MenuServiceImpl.delete가 실행되면 IndexingEvent(DELETE, MENU, idx)가 발행되고 검색 결과에서 제거된다
4. **AC4**: 콘텐츠의 공개/비공개 상태가 변경될 때 visibility가 변경되면 IndexingEvent(UPDATE)가 발행되고 ES 문서의 visibility 필드가 갱신된다
5. **AC5**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: CMS 서비스 분석
  - [x] 1.1 DynamicBoardServiceImpl 구조 파악 (save/delete)
  - [x] 1.2 ContentServiceImpl 구조 파악 (create*/update*/delete*)
  - [x] 1.3 MenuServiceImpl 구조 파악 (saveDriveMenu/deleteDriveAndAllChildren)

- [x] Task 2: 이벤트 발행 통합
  - [x] 2.1 DynamicBoardServiceImpl에 ApplicationEventPublisher 주입 및 이벤트 발행
  - [x] 2.2 ContentServiceImpl에 이벤트 발행
  - [x] 2.3 MenuServiceImpl에 이벤트 발행

- [x] Task 3: 빌드 검증 (AC: #5)
  - [x] 3.1 gradle compileJava + test 성공

## Dev Notes

### 최소 침습 원칙

기존 CMS Service에 1줄 추가로 이벤트 발행:
```java
// 기존 코드
boardRepository.save(post);

// 추가 1줄
applicationEventPublisher.publishEvent(IndexingEvent.postCreated(this, post.getIdx()));
```

### 이벤트 발행 위치

- **CREATE**: save() 직후
- **UPDATE**: save() 직후
- **DELETE**: delete() 직후
- **Visibility 변경**: update() 시 visibility 필드 변경 감지 후 UPDATE 이벤트

### 구현 상세

#### DynamicBoardServiceImpl (POST)
- `save()`: CREATE 시 `postCreated(newIdx)`, UPDATE 시 `postUpdated(idx)`
- `delete()`: `postDeleted(idx)`
- 생성 시 idx 반환 위해 `DynamicBoardDao.insertByMenuId()` 반환 타입을 `Long`으로 변경
- `KeyHolder` 사용하여 generated key 반환

#### ContentServiceImpl (PAGE)
- `createRootContent()`: `pageCreated(saved.getIdx())`
- `createChildContent()`: `pageCreated(saved.getIdx())`
- `updateContent()`: `pageUpdated(idx)`
- `activeContent()`: `pageUpdated(idx)` (is_main 변경도 UPDATE 이벤트)
- `deleteContentByIdx()`: `pageDeleted(idx)`
- `deleteContentByParentId()`: 그룹 전체 삭제 시 각 idx에 대해 `pageDeleted(idx)` 발행

#### MenuServiceImpl (MENU)
- `saveDriveMenu()`: CREATE 시 `menuCreated(id)`, UPDATE 시 `menuUpdated(id)`
- `deleteDriveAndAllChildren()`: 드라이브 및 자손 메뉴 각각에 대해 `menuDeleted(id)` 발행

### 변경된 파일

1. `DynamicBoardDao.java` - `insertByMenuId` 반환 타입 `Long`으로 변경
2. `DynamicBoardDaoImpl.java` - `KeyHolder` 사용하여 generated key 반환
3. `DynamicBoardServiceImpl.java` - `ApplicationEventPublisher` 주입, 이벤트 발행
4. `ContentServiceImpl.java` - `ApplicationEventPublisher` 주입, 이벤트 발행
5. `MenuServiceImpl.java` - `ApplicationEventPublisher` 주입, 이벤트 발행
