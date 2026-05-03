# Data Model: CLI TODO Core Management

**Feature**: CLI TODO Core Management | **Date**: 2026-05-03
**Phase**: 1 (Design & Architecture)

## Overview

CLI TODO 앱의 데이터 모델을 설계한다. TodoItem 엔티티를 중심으로 한 단순하고 확장 가능한 구조를採用한다.

**Design Principles**:
- **단순함 우선**: 단일 TodoItem 엔티티로 모든 요구사항 충족
- **확장성**: 향후 기능 추가 시 쉽게 확장 가능
- **성능**: SQLite에 최적화된 인덱스 설계
- **무결성**: 데이터베이스 레벨에서 제약조건 적용

## Entity: TodoItem

### Core Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | INTEGER | ✅ | AUTO_INCREMENT | 고유 식별자 (1부터 시작하는 정수) |
| `title` | VARCHAR(256) | ✅ | - | TODO 항목 제목 |
| `status` | VARCHAR(20) | ✅ | 'incomplete' | 항목 상태 ('incomplete', 'complete') |
| `priority` | VARCHAR(20) | ✅ | 'medium' | 우선순위 ('low', 'medium', 'high') |
| `due_date` | DATE | ❌ | NULL | 마감일 (YYYY-MM-DD 형식) |
| `created_at` | DATETIME | ✅ | CURRENT_TIMESTAMP | 생성 시간 |
| `updated_at` | DATETIME | ✅ | CURRENT_TIMESTAMP | 마지막 수정 시간 |

### Field Constraints

#### id
- **Primary Key**: 자동 증가 정수
- **Range**: 1 ~ 2^63-1 (SQLite INTEGER 범위)
- **Uniqueness**: 데이터베이스 레벨에서 보장

#### title
- **Required**: 빈 문자열 불허용
- **Length**: 1-256자 (대부분의 TODO 제목 충분)
- **Encoding**: UTF-8 (한글 지원)
- **Trimming**: 앞뒤 공백 자동 제거

#### status
- **Enum Values**: 'incomplete', 'complete'
- **Default**: 'incomplete'
- **Validation**: 허용된 값만 입력 가능

#### priority
- **Enum Values**: 'low', 'medium', 'high'
- **Default**: 'medium'
- **Ordering**: high > medium > low (숫자 변환 시 3 > 2 > 1)

#### due_date
- **Format**: ISO 8601 날짜 형식 (YYYY-MM-DD)
- **Optional**: NULL 허용
- **Validation**: 유효한 날짜 형식 검증
- **Future Dates**: 과거 날짜도 허용 (완료된 작업 추적용)

#### created_at, updated_at
- **Format**: ISO 8601 datetime (YYYY-MM-DDTHH:MM:SSZ)
- **Timezone**: UTC (표준시)
- **Auto-update**: updated_at은 레코드 수정 시 자동 갱신

## Database Schema

### Table: todos

```sql
CREATE TABLE todos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(256) NOT NULL CHECK(length(title) > 0),
    status VARCHAR(20) NOT NULL DEFAULT 'incomplete'
        CHECK(status IN ('incomplete', 'complete')),
    priority VARCHAR(20) NOT NULL DEFAULT 'medium'
        CHECK(priority IN ('low', 'medium', 'high')),
    due_date DATE NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Indexes

```sql
-- 상태별 조회 최적화
CREATE INDEX idx_todos_status ON todos(status);

-- 우선순위별 조회 최적화
CREATE INDEX idx_todos_priority ON todos(priority);

-- 생성 시간 역순 조회 최적화 (최신 항목 우선)
CREATE INDEX idx_todos_created_at ON todos(created_at DESC);

-- 복합 인덱스: 상태 + 우선순위 필터링
CREATE INDEX idx_todos_status_priority ON todos(status, priority);

-- 마감일 조회 (있는 경우만)
CREATE INDEX idx_todos_due_date ON todos(due_date) WHERE due_date IS NOT NULL;
```

## Data Validation Rules

### Business Rules

1. **Title Uniqueness**: 동일 제목 허용 (중복 TODO 가능)
2. **Status Transition**: incomplete ↔ complete 자유 전환
3. **Priority Levels**: 3단계 (low, medium, high)로 제한
4. **Due Date**: 미래/과거 날짜 모두 허용
5. **Timestamps**: 생성 시각은 불변, 수정 시각은 자동 갱신

### Input Validation

#### Title Validation
```python
def validate_title(title: str) -> str:
    if not title or not title.strip():
        raise ValueError("제목은 필수입니다")
    if len(title.strip()) > 256:
        raise ValueError("제목은 256자를 초과할 수 없습니다")
    return title.strip()
```

#### Status Validation
```python
def validate_status(status: str) -> str:
    valid_statuses = {'incomplete', 'complete'}
    if status not in valid_statuses:
        raise ValueError(f"상태는 {valid_statuses} 중 하나여야 합니다")
    return status
```

#### Priority Validation
```python
def validate_priority(priority: str) -> str:
    valid_priorities = {'low', 'medium', 'high'}
    if priority not in valid_priorities:
        raise ValueError(f"우선순위는 {valid_priorities} 중 하나여야 합니다")
    return priority
```

#### Due Date Validation
```python
def validate_due_date(due_date: str | None) -> str | None:
    if due_date is None:
        return None
    try:
        # YYYY-MM-DD 형식 검증
        datetime.fromisoformat(due_date)
        return due_date
    except ValueError:
        raise ValueError("마감일은 YYYY-MM-DD 형식이어야 합니다")
```

## Data Relationships

### Entity Relationships

```
TodoItem
├── id: Primary Key
├── title: Core Data
├── status: State
├── priority: Metadata
├── due_date: Optional Metadata
├── created_at: Audit
└── updated_at: Audit
```

**Relationship Type**: 단일 엔티티 (관계 없음)
**Justification**: TODO 앱의 단순한 도메인 특성상 관계형 모델 불필요

## Migration Strategy

### Initial Schema (v1.0.0)

```sql
-- 초기 스키마 생성
CREATE TABLE todos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title VARCHAR(256) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'incomplete',
    priority VARCHAR(20) NOT NULL DEFAULT 'medium',
    due_date DATE,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- 기본 인덱스 생성
CREATE INDEX idx_todos_status ON todos(status);
CREATE INDEX idx_todos_priority ON todos(priority);
CREATE INDEX idx_todos_created_at ON todos(created_at DESC);
```

### Future Migration Path

**v1.1.0**: 제약조건 추가
```sql
-- CHECK 제약조건 추가
ALTER TABLE todos ADD CONSTRAINT chk_title_length CHECK(length(title) > 0);
ALTER TABLE todos ADD CONSTRAINT chk_status_valid CHECK(status IN ('incomplete', 'complete'));
ALTER TABLE todos ADD CONSTRAINT chk_priority_valid CHECK(priority IN ('low', 'medium', 'high'));
```

**v1.2.0**: 복합 인덱스 추가 (성능 최적화)
```sql
CREATE INDEX idx_todos_status_priority ON todos(status, priority);
CREATE INDEX idx_todos_due_date ON todos(due_date) WHERE due_date IS NOT NULL;
```

## Performance Considerations

### Query Optimization

#### Common Queries

1. **전체 목록 조회** (생성 시간 역순)
   ```sql
   SELECT * FROM todos ORDER BY created_at DESC;
   -- Index: idx_todos_created_at 활용
   ```

2. **상태별 필터링**
   ```sql
   SELECT * FROM todos WHERE status = ? ORDER BY created_at DESC;
   -- Index: idx_todos_status 활용
   ```

3. **우선순위별 필터링**
   ```sql
   SELECT * FROM todos WHERE priority = ? ORDER BY created_at DESC;
   -- Index: idx_todos_priority 활용
   ```

4. **복합 필터링**
   ```sql
   SELECT * FROM todos WHERE status = ? AND priority = ? ORDER BY created_at DESC;
   -- Index: idx_todos_status_priority 활용
   ```

#### Performance Targets

- **Insert**: 1000개 항목 일괄 삽입 시 500ms 이내
- **Select All**: 1000개 항목 조회 시 200ms 이내
- **Select Filtered**: 1000개 중 100개 필터링 시 150ms 이내
- **Update**: 단일 항목 수정 시 5ms 이내
- **Delete**: 단일 항목 삭제 시 5ms 이내

### Storage Estimation

#### Record Size Calculation

- **Fixed Fields**: id(8) + created_at(8) + updated_at(8) = 24 bytes
- **Variable Fields**: title(평균 50자) + status(10) + priority(6) + due_date(10) = ~76 bytes
- **Overhead**: SQLite 레코드 오버헤드 ~20 bytes
- **Total per Record**: ~120 bytes

#### Scale Estimates

- **1000개 항목**: ~120KB (메모리 적재 가능)
- **10000개 항목**: ~1.2MB (여전히 메모리 적재 가능)
- **100000개 항목**: ~12MB (디스크 기반 처리 권장)

## Data Integrity & Backup

### Integrity Checks

1. **Foreign Key Constraints**: 없음 (단일 테이블)
2. **Check Constraints**: 필드 값 범위 검증
3. **Not Null Constraints**: 필수 필드 보장
4. **Unique Constraints**: 없음 (중복 제목 허용)

### Backup Strategy

1. **File-level Backup**: `~/.todo/todos.db` 파일 복사
2. **Export Feature**: JSON 형식으로 데이터 내보내기
3. **Recovery**: 손상된 DB 파일 복구 기능

### Data Export Format

```json
{
  "version": "1.0",
  "exported_at": "2026-05-03T10:00:00Z",
  "todos": [
    {
      "id": 1,
      "title": "프로젝트 계획서 작성",
      "status": "incomplete",
      "priority": "high",
      "due_date": "2026-05-10",
      "created_at": "2026-05-03T09:00:00Z",
      "updated_at": "2026-05-03T09:00:00Z"
    }
  ]
}
```

## Testing Strategy

### Unit Tests

1. **Model Validation**: 각 필드의 유효성 검증
2. **Database Operations**: CRUD 작업 단위 테스트
3. **Index Performance**: 쿼리 실행 계획 검증

### Integration Tests

1. **Full CRUD Cycle**: 생성 → 조회 → 수정 → 삭제
2. **Concurrent Access**: 다중 프로세스 동시 접근 테스트
3. **Data Migration**: 스키마 변경 시 데이터 보존 검증

### Performance Tests

1. **Load Testing**: 1000개 항목 CRUD 성능 측정
2. **Memory Usage**: 다양한 데이터 크기에서의 메모리 사용량
3. **Query Optimization**: 인덱스 효과성 검증

## Conclusion

설계된 데이터 모델은 요구사항을 완전히 충족하면서도 단순성과 확장성을 모두 고려하였다.

**Key Strengths**:
- ✅ **단순함**: 단일 테이블로 모든 기능 구현
- ✅ **성능**: 적절한 인덱스로 쿼리 최적화
- ✅ **무결성**: 데이터베이스 레벨 제약조건
- ✅ **확장성**: 미래 기능 추가를 위한 구조적 기반

**Ready for Implementation**: Phase 2 구현으로 진행 가능

---

**Data Model Version**: 1.0 | **Designed**: 2026-05-03 | **Designer**: Data Architecture Team