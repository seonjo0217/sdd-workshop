# Feature Specification: CLI TODO Core Management

**Feature Branch**: `001-cli-todo-core`  
**Created**: 2026-05-03  
**Status**: Draft  
**Input**: User description: "cli 기반의 todo 관리 앱을 만들고 싶어요. 사용자: 터미널에서 사용하는 개인 개발자. 주요기능: 1) todo 항목 추가, 제목(필수), 마감일(선택), 우선순위(선택) 2) 전체 목록 조회:완료/미완료/우선순위로 필터링 가능 3) 항목 완료 처리: 항목ID로 완료 표시 4) 항목 삭제 처리: 항목 ID로 삭제"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Add TODO Items with Metadata (Priority: P1)

개인 개발자가 새로운 TODO 항목을 빠르게 추가할 수 있어야 하며, 제목은 필수 정보이고 마감일과 우선순위는 선택적으로 지정할 수 있어야 한다. 이를 통해 업무의 우선순위를 관리하고 마감기한을 추적할 수 있다.

**Why this priority**: 이 기능은 TODO 앱의 핵심 가치 제공의 시작점이며, 다른 모든 기능(조회, 완료 처리, 삭제)은 이 기능으로 생성된 항목을 기반으로 동작한다.

**Independent Test**: 단일 TODO 항목을 제목만으로 추가하면 앱이 정상 작동하고, 선택적 필드 없이도 완전한 기능을 제공한다. 이를 통해 P1 기능만으로도 최소 가용 제품(MVP)이 성립한다.

**Acceptance Scenarios**:

1. **Given** 앱이 시작된 상태, **When** 사용자가 `add` 명령어로 제목만 입력, **Then** 새 항목이 생성되고 고유 ID가 할당됨
2. **Given** 기존 TODO 항목이 있는 상태, **When** 사용자가 `add` 명령어로 제목, 마감일, 우선순위를 입력, **Then** 모든 정보를 포함한 항목이 생성됨
3. **Given** `add` 명령어 실행 상태, **When** 사용자가 제목 없이 입력, **Then** 오류 메시지가 표시되고 항목이 생성되지 않음

---

### User Story 2 - View and Filter TODO List (Priority: P1)

개인 개발자가 현재 업무 상태를 파악하기 위해 모든 TODO 항목을 조회하고, 완료/미완료 상태, 우선순위로 필터링할 수 있어야 한다.

**Why this priority**: 이 기능은 P1 "Add TODO Items"과 동등하게 중요하다. 추가된 항목을 확인할 수 없으면 앱의 가치가 없기 때문이다.

**Independent Test**: 전체 목록 조회 기능만 독립적으로 테스트 가능하며, P1 기능(Add)과 함께 완성된 MVP를 제공한다.

**Acceptance Scenarios**:

1. **Given** 여러 TODO 항목이 저장된 상태, **When** 사용자가 `list` 명령어 실행, **Then** 모든 항목이 표시되고 각 항목의 ID, 제목, 상태가 보임
2. **Given** 완료된 항목과 미완료 항목이 섞여 있는 상태, **When** 사용자가 `list --status=incomplete` 실행, **Then** 미완료 항목만 표시됨
3. **Given** 다양한 우선순위의 항목이 있는 상태, **When** 사용자가 `list --priority=high` 실행, **Then** 높은 우선순위 항목만 표시됨
4. **Given** 필터 조건이 없는 상태, **When** 사용자가 `list`로 전체 목록 조회, **Then** 모든 항목이 연대순(생성순 역순) 또는 우선순위 순으로 정렬되어 표시됨

---

### User Story 3 - Mark TODO Items as Complete (Priority: P2)

개인 개발자가 완료한 업무를 표시하여 진행 상태를 추적하고, 미완료 항목에 집중할 수 있어야 한다.

**Why this priority**: 이 기능은 업무 진행 상황 추적을 가능하게 하지만, P1 (Add + List) 없이는 의미가 없으므로 P2이다.

**Independent Test**: "완료 표시" 기능은 독립적으로 테스트 가능하며, P1 + P2 기능으로 기본적인 TODO 앱 기능이 완성된다.

**Acceptance Scenarios**:

1. **Given** 미완료 상태의 TODO 항목이 있는 상태, **When** 사용자가 항목 ID로 `done <ID>` 명령어 실행, **Then** 해당 항목이 완료 상태로 변경됨
2. **Given** 이미 완료된 항목, **When** 사용자가 다시 `done <ID>` 실행, **Then** 항목이 미완료 상태로 되돌려짐 (토글 기능)
3. **Given** 존재하지 않는 ID, **When** 사용자가 `done <invalid-ID>` 실행, **Then** 오류 메시지가 표시되고 기존 항목은 변경되지 않음

---

### User Story 4 - Delete TODO Items (Priority: P2)

개인 개발자가 불필요하거나 중복된 TODO 항목을 삭제하여 목록을 정리할 수 있어야 한다.

**Why this priority**: 이 기능은 선택적이며, 필수 CRUD 기능 중 마지막이다. P1 기능(Add + List) 없이는 의미가 없으므로 P2이다.

**Independent Test**: "삭제" 기능은 독립적으로 테스트 가능하며, P1 + P2-delete로 완전한 CRUD 앱이 된다.

**Acceptance Scenarios**:

1. **Given** 여러 TODO 항목이 있는 상태, **When** 사용자가 `delete <ID>` 명령어 실행, **Then** 해당 항목이 제거되고 목록에서 사라짐
2. **Given** 이미 삭제된 또는 존재하지 않는 항목 ID, **When** 사용자가 `delete <invalid-ID>` 실행, **Then** 오류 메시지가 표시되고 다른 항목은 영향을 받지 않음
3. **Given** 삭제 대상 항목 확인이 필요한 상황, **When** 사용자가 `delete <ID> --confirm` 실행, **Then** 항목이 삭제됨 (또는 확인 프롬프트 표시 후 삭제)

---

## Functional Requirements

### R1: TODO Item Creation
- 사용자는 CLI 명령어 `add <title>` 으로 새 TODO 항목을 추가할 수 있다
- 제목(title)은 필수 정보이며, 1자 이상 256자 이하의 문자열이어야 한다
- 마감일(due_date)은 ISO 8601 형식(YYYY-MM-DD)의 선택적 정보이다
- 우선순위(priority)는 'low', 'medium', 'high' 중 하나의 선택적 정보이다 (기본값: 'medium')
- 각 항목은 고유 ID(숫자 또는 UUID)를 자동으로 할당받는다
- 생성 시간(created_at)은 자동으로 기록된다

### R2: TODO Item Retrieval
- 사용자는 `list` 명령어로 모든 TODO 항목을 조회할 수 있다
- 각 항목 표시에는 ID, 제목, 상태(완료/미완료), 우선순위, 마감일(있는 경우)이 포함된다
- 기본 정렬 순서는 생성 시간 역순(최신 항목 먼저) 또는 우선순위 순이다
- `--status=complete|incomplete` 옵션으로 상태별 필터링이 가능하다
- `--priority=low|medium|high` 옵션으로 우선순위별 필터링이 가능하다
- 필터는 조합 가능하다 (예: `list --status=incomplete --priority=high`)
- JSON 출력 옵션 지원으로 자동화 도구와의 통합을 가능하게 한다

### R3: TODO Item Status Update
- 사용자는 `done <ID>` 명령어로 항목의 완료 상태를 전환할 수 있다 (미완료 → 완료 또는 완료 → 미완료)
- 상태 변경 시 마지막 수정 시간(updated_at)이 기록된다
- 유효하지 않은 ID로 상태 변경을 시도하면 명확한 오류 메시지를 표시한다

### R4: TODO Item Deletion
- 사용자는 `delete <ID>` 명령어로 TODO 항목을 삭제할 수 있다
- 삭제된 항목은 복구할 수 없다
- 삭제 전 확인 프롬프트를 표시하거나 `--force` 옵션으로 즉시 삭제할 수 있다
- 유효하지 않은 ID로 삭제를 시도하면 오류 메시지를 표시한다

### R5: Data Persistence
- 모든 TODO 항목은 로컬 파일 시스템에 저장된다 (JSON 형식 권장)
- 저장 파일 위치는 사용자 홈 디렉토리 또는 앱 설정 디렉토리에 위치한다 (예: `~/.config/todo/` 또는 `~/.todo/`)
- 데이터 손상 시에도 앱이 우아하게 실패하고 명확한 오류 메시지를 제공한다

### R6: Error Handling
- 모든 오류는 stderr로 출력된다
- 오류 메시지는 사용자가 이해할 수 있는 명확한 한국어로 작성된다
- 오류 발생 시 적절한 exit code를 반환한다 (0=성공, 1=일반 오류, 2=사용법 오류)

### R7: Help and Usage Information
- 사용자는 `--help` 또는 `-h` 옵션으로 전체 앱 또는 특정 명령어의 도움말을 볼 수 있다
- 도움말에는 사용 가능한 명령어, 옵션, 예제가 포함된다

---

## Success Criteria

1. **기능 완성도**: 4개 주요 기능(Add, List+Filter, Done, Delete)이 모두 구현되고 테스트됨 → **테스트 커버리지 80% 이상**
2. **사용 용이성**: 개인 개발자가 5분 내에 앱을 설치하고 첫 번째 TODO를 추가 및 조회할 수 있음 → **설치 시간 < 5분, 학습곡선 < 3개 예제**
3. **데이터 안정성**: 1000개 TODO 항목을 저장하고 조회할 때 응답 시간이 1초 이내 → **조회 성능 < 1초 (1000개 데이터셋)**
4. **오류 처리**: 모든 사용자 오류 입력에 대해 명확한 오류 메시지 제공 → **오류 메시지 적중률 100%**
5. **확장성**: 비즈니스 로직과 CLI 인터페이스가 완전히 분리되어 향후 다른 인터페이스(GUI, API) 추가 가능 → **의존성 수 ≤ 3**
6. **자동화 가능성**: JSON 출력 형식을 지원하여 다른 CLI 도구와 파이프라인으로 연결 가능 → **`list --json` 명령어 구현**

---

## Key Entities

### Entity: TodoItem
- **id** (integer | UUID): 고유 식별자, 자동 생성
- **title** (string): 항목 제목, 필수, 1-256 자
- **status** (enum: 'incomplete' | 'complete'): 현재 상태, 기본값 'incomplete'
- **priority** (enum: 'low' | 'medium' | 'high'): 우선순위, 기본값 'medium'
- **due_date** (date | null): 마감일, ISO 8601 형식, 선택적
- **created_at** (datetime): 생성 시간, 자동 기록
- **updated_at** (datetime): 마지막 수정 시간, 자동 기록

### Storage Format (JSON 권장)
```json
{
  "todos": [
    {
      "id": 1,
      "title": "프로젝트 계획서 작성",
      "status": "incomplete",
      "priority": "high",
      "due_date": "2026-05-10",
      "created_at": "2026-05-03T14:30:00Z",
      "updated_at": "2026-05-03T14:30:00Z"
    }
  ]
}
```

---

## Assumptions

1. **사용자 환경**: Python 3.8+ 또는 Go, Node.js 등의 환경 가용 (기술스택 미정이므로 표준 환경 가정)
2. **파일 시스템 접근**: 사용자의 홈 디렉토리에 쓰기 권한 있음
3. **싱글 유저**: 앱은 한 번에 한 사용자만 사용하며, 동시성 문제 없음
4. **로컬 저장소**: 클라우드 동기화, 네트워크 저장소 미지원 (Constitution Principle V)
5. **기본값 제공**: 우선순위 기본값 'medium', 마감일 없음으로 기본값 제공

---

## Out of Scope

- REST API 서버 구현
- 웹 사용자 인터페이스 (GUI)
- 데이터베이스 서버 (로컬 파일 시스템만 사용)
- 클라우드 동기화 또는 온라인 기능
- 다중 사용자 지원
- 팀 협업 기능
