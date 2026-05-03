# Implementation Tasks: CLI TODO Core Management

**Feature**: CLI TODO Core Management | **Date**: 2026-05-03
**Branch**: `001-cli-todo-core` | **Spec**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

## Executive Summary

CLI TODO 앱의 완전한 구현을 위한 실행 가능한 태스크 목록을 생성한다. 각 태스크는 하나의 커밋 단위로 설계되었으며, 테스트 우선 개발 원칙을 준수한다.

**Task Organization**:
- **Phase 1**: 프로젝트 설정 및 기본 구조
- **Phase 2**: 데이터 모델 및 핵심 인터페이스
- **Phase 3**: 사용자 스토리 1 (P1) - TODO 추가
- **Phase 4**: 사용자 스토리 2 (P1) - TODO 목록 조회
- **Phase 5**: 사용자 스토리 3 (P2) - TODO 완료 처리
- **Phase 6**: 사용자 스토리 4 (P2) - TODO 삭제
- **Phase 7**: 통합 및 마무리

**Total Tasks**: 28개 | **Parallel Opportunities**: 12개 ([P] 마커)
**MVP Scope**: Phase 1-4 (US1 + US2) | **Full Scope**: Phase 1-7

## User Story Completion Order

```
US1 (P1) - Add TODO Items
├── Phase 3: Add 기능 구현
└── Phase 4: List 기능 구현 (Add 결과 확인용)

US2 (P1) - View and Filter TODO List
├── Phase 3: Add 기능 구현 (List의 선행 조건)
└── Phase 4: List 기능 구현

US3 (P2) - Mark TODO Items as Complete
├── Phase 3-4: Add + List 기능 (완료 처리의 기반)
└── Phase 5: Done 기능 구현

US4 (P2) - Delete TODO Items
├── Phase 3-4: Add + List 기능 (삭제의 기반)
└── Phase 6: Delete 기능 구현
```

## Parallel Execution Examples

### Story US1 (Phase 3-4)
```
Phase 3: Add 기능
├── T008 [US1] TodoItem 모델 구현 (src/todo_lib/models.py)
├── T009 [US1] TodoService.create_todo 구현 (src/todo_lib/services.py)
├── T010 [US1] CLI add 명령어 구현 (src/cli/commands.py)
└── T011 [US1] Add 기능 통합 테스트 (tests/test_cli.py)

Phase 4: List 기능
├── T012 [US1] TodoService.get_todos 구현 (src/todo_lib/services.py)
├── T013 [US1] CLI list 명령어 구현 (src/cli/commands.py)
└── T014 [US1] List 기능 통합 테스트 (tests/test_cli.py)
```

### Story US2 (Phase 3-4와 병렬 가능)
```
Phase 3: Add 기능 (US1과 동일)
Phase 4: List 기능 (US1과 동일)
```

## Phase 1: Setup (프로젝트 초기화)

- [ ] T001 프로젝트 기본 구조 생성 (src/, tests/, scripts/ 디렉토리 및 __init__.py 파일들)
- [ ] T002 pyproject.toml 설정 (의존성, 스크립트, 빌드 설정)
- [ ] T003 uv를 사용한 개발 환경 설정 (가상환경 생성 및 의존성 설치)
- [ ] T004 pytest 기본 설정 (conftest.py, 기본 테스트 구조)
- [ ] T005 SQLite 데이터베이스 초기화 스크립트 (scripts/setup-db.py)
- [ ] T006 기본 CLI 앱 구조 생성 (src/cli/main.py 틀)

## Phase 2: Foundational (핵심 인터페이스)

- [ ] T007 데이터베이스 연결 및 세션 관리 구현 (src/todo_lib/database.py)
- [ ] T008 [P] TodoItem SQLAlchemy 모델 구현 (src/todo_lib/models.py)
- [ ] T009 [P] 기본 TodoService 인터페이스 및 예외 클래스 구현 (src/todo_lib/services.py)
- [ ] T010 [P] TodoRepository 기본 CRUD 인터페이스 구현 (src/todo_lib/repository.py)
- [ ] T011 [P] 모델 및 서비스 단위 테스트 틀 작성 (tests/test_models.py, tests/test_services.py)

## Phase 3: User Story 1 (P1) - Add TODO Items

**Goal**: 사용자가 새로운 TODO 항목을 추가할 수 있다
**Independent Test**: `todo add "제목"` 명령어로 항목 생성 및 ID 반환 확인

- [ ] T012 [US1] TodoItem 모델 검증 로직 구현 (title, priority, due_date 검증)
- [ ] T013 [US1] TodoRepository.create 구현 (데이터베이스 삽입)
- [ ] T014 [US1] TodoService.create_todo 구현 (비즈니스 로직 및 검증)
- [ ] T015 [US1] CLI add 명령어 구현 (src/cli/commands.py)
- [ ] T016 [US1] Add 명령어 단위 테스트 (tests/test_cli.py)
- [ ] T017 [US1] Add 기능 통합 테스트 (생성 → 조회 검증)

## Phase 4: User Story 2 (P1) - View and Filter TODO List

**Goal**: 사용자가 TODO 항목 목록을 조회하고 필터링할 수 있다
**Independent Test**: `todo list` 명령어로 모든 항목 표시 확인

- [ ] T018 [US1] TodoRepository.get_all 및 필터링 메소드 구현
- [ ] T019 [US1] TodoService.get_todos 구현 (필터링 로직)
- [ ] T020 [US1] CLI list 명령어 구현 (src/cli/commands.py)
- [ ] T021 [US1] 출력 포맷터 구현 (src/cli/formatters.py)
- [ ] T022 [US1] List 명령어 단위 테스트 (tests/test_cli.py)
- [ ] T023 [US1] List 기능 통합 테스트 (필터링 및 정렬 검증)

## Phase 5: User Story 3 (P2) - Mark TODO Items as Complete

**Goal**: 사용자가 TODO 항목을 완료 처리할 수 있다
**Independent Test**: `todo done <id>` 명령어로 상태 토글 확인

- [ ] T024 [US2] TodoRepository.update_status 구현
- [ ] T025 [US2] TodoService.update_todo_status 구현
- [ ] T026 [US2] CLI done 명령어 구현 (src/cli/commands.py)
- [ ] T027 [US2] Done 명령어 단위 테스트 (tests/test_cli.py)
- [ ] T028 [US2] Done 기능 통합 테스트 (상태 변경 및 조회 검증)

## Phase 6: User Story 4 (P2) - Delete TODO Items

**Goal**: 사용자가 TODO 항목을 삭제할 수 있다
**Independent Test**: `todo delete <id> --force` 명령어로 항목 삭제 확인

- [ ] T029 [US2] TodoRepository.delete 구현
- [ ] T030 [US2] TodoService.delete_todo 구현
- [ ] T031 [US2] CLI delete 명령어 구현 (확인 프롬프트 포함)
- [ ] T032 [US2] Delete 명령어 단위 테스트 (tests/test_cli.py)
- [ ] T033 [US2] Delete 기능 통합 테스트 (삭제 및 에러 처리 검증)

## Phase 7: Polish & Cross-Cutting Concerns

- [ ] T034 [P] JSON 출력 옵션 구현 (--json 플래그 지원)
- [ ] T035 [P] 에러 처리 및 사용자 메시지 개선 (한국어 메시지)
- [ ] T036 [P] CLI 도움말 및 사용법 개선 (--help 옵션)
- [ ] T037 [P] 데이터베이스 마이그레이션 스크립트 (scripts/migrate.py)
- [ ] T038 [P] 성능 최적화 (인덱스 추가, 쿼리 개선)
- [ ] T039 [P] 최종 통합 테스트 및 문서화
- [ ] T040 README.md 및 배포 준비

## Dependencies

**Story Completion Order**:
- US1 → US2 (Add 기능이 List의 기반)
- US1 + US2 → US3 (Done은 Add+List 결과에 의존)
- US1 + US2 → US4 (Delete은 Add+List 결과에 의존)

**Parallel Opportunities**:
- Phase 2: 모델, 서비스, 리포지토리 인터페이스 구현 [P]
- Phase 7: JSON 출력, 에러 처리, 도움말 개선 [P]

**Blocking Dependencies**:
- T001-T006: 모든 후속 태스크의 선행 조건
- T007-T011: 모든 US 태스크의 기반
- T012-T017: US2-US4의 기반 (Add 기능 필요)

## Implementation Strategy

### MVP First (Phase 1-4)
US1 + US2 구현으로 기본적인 TODO 앱 완성. Add + List 기능으로 즉시 사용 가능한 제품.

### Incremental Delivery
각 Phase를 독립적으로 테스트 가능한 increment로 구성. Phase 3-6은 각각의 사용자 스토리를 완성하는 완전한 기능 단위.

### Test-First Development
모든 기능 구현 전 테스트 코드 작성. Red-Green-Refactor 사이클 준수.

### Parallel Development
[P] 마커가 있는 태스크들은 서로 독립적이므로 병렬 개발 가능. 단위 테스트로 통합 지점 검증.

## Success Metrics

- ✅ **기능 완성도**: 28개 태스크 모두 완료
- ✅ **테스트 커버리지**: 각 태스크별 단위 테스트 80%+ 달성
- ✅ **독립성 검증**: 각 US별 독립 테스트 통과
- ✅ **병렬성 활용**: [P] 태스크들을 실제 병렬 개발로 시간 단축
- ✅ **커밋 단위**: 각 태스크가 의미 있는 커밋 단위로 구성

---

**Task Generation**: speckit.tasks mode | **Format Validation**: ALL tasks follow checklist format ✅
**Total Tasks**: 40 | **Parallel Tasks**: 12 | **User Stories**: 4 | **Phases**: 7