# Service Interface Contracts

**Feature**: CLI TODO Core Management | **Date**: 2026-05-03
**Phase**: 1 (Design & Architecture)

## Overview

비즈니스 로직 계층의 서비스 인터페이스를 정의한다. CLI와 데이터 액세스 계층 사이의 계약을 명세하여 레이어 분리 원칙을 준수한다.

**Interface Principles**:
- **단순함**: 최소한의 메소드로 모든 기능 제공
- **일관성**: 모든 메소드가 동일한 패턴 준수
- **예외 처리**: 명확한 에러 타입과 메시지
- **타입 안전**: Python 타입 힌트를 활용한 인터페이스 정의

## Service Interface: TodoService

### Core Methods

#### 1. create_todo

**Signature**:
```python
def create_todo(
    self,
    title: str,
    priority: str = "medium",
    due_date: Optional[str] = None
) -> TodoItem:
    """
    새 TODO 항목을 생성한다.

    Args:
        title: TODO 제목 (필수, 1-256자)
        priority: 우선순위 ("low", "medium", "high")
        due_date: 마감일 (선택, "YYYY-MM-DD" 형식)

    Returns:
        생성된 TodoItem 객체

    Raises:
        ValidationError: 입력 값이 유효하지 않은 경우
        DatabaseError: 데이터베이스 작업 실패 시
    """
```

**Input Validation**:
- `title`: 필수, 공백 제거 후 1-256자
- `priority`: "low", "medium", "high" 중 하나
- `due_date`: None 또는 유효한 YYYY-MM-DD 형식

**Output Contract**:
- 새로 생성된 TodoItem 객체 반환
- id, created_at, updated_at 자동 설정
- status는 "incomplete"로 초기화

#### 2. get_todos

**Signature**:
```python
def get_todos(
    self,
    status: Optional[str] = None,
    priority: Optional[str] = None
) -> List[TodoItem]:
    """
    TODO 항목 목록을 조회한다.

    Args:
        status: 상태 필터 ("incomplete", "complete")
        priority: 우선순위 필터 ("low", "medium", "high")

    Returns:
        필터링된 TodoItem 객체 리스트 (생성 시간 역순)

    Raises:
        DatabaseError: 데이터베이스 조회 실패 시
    """
```

**Input Validation**:
- `status`: None 또는 "incomplete", "complete"
- `priority`: None 또는 "low", "medium", "high"

**Output Contract**:
- List[TodoItem] 반환
- 생성 시간(created_at) 역순 정렬
- 필터 조건에 따라 결과 제한

#### 3. get_todo_by_id

**Signature**:
```python
def get_todo_by_id(self, todo_id: int) -> TodoItem:
    """
    ID로 특정 TODO 항목을 조회한다.

    Args:
        todo_id: TODO 항목 ID (양의 정수)

    Returns:
        TodoItem 객체

    Raises:
        NotFoundError: 해당 ID의 항목이 없는 경우
        ValidationError: ID 형식이 잘못된 경우
        DatabaseError: 데이터베이스 조회 실패 시
    """
```

**Input Validation**:
- `todo_id`: 양의 정수

**Output Contract**:
- 단일 TodoItem 객체 반환
- 존재하지 않는 ID는 NotFoundError 발생

#### 4. update_todo_status

**Signature**:
```python
def update_todo_status(self, todo_id: int, status: str) -> TodoItem:
    """
    TODO 항목의 상태를 변경한다.

    Args:
        todo_id: TODO 항목 ID
        status: 새 상태 ("incomplete", "complete")

    Returns:
        업데이트된 TodoItem 객체

    Raises:
        NotFoundError: 해당 ID의 항목이 없는 경우
        ValidationError: 입력 값이 유효하지 않은 경우
        DatabaseError: 데이터베이스 업데이트 실패 시
    """
```

**Input Validation**:
- `todo_id`: 양의 정수, 존재하는 ID
- `status`: "incomplete" 또는 "complete"

**Output Contract**:
- 업데이트된 TodoItem 객체 반환
- updated_at 필드 자동 갱신

#### 5. delete_todo

**Signature**:
```python
def delete_todo(self, todo_id: int) -> None:
    """
    TODO 항목을 삭제한다.

    Args:
        todo_id: 삭제할 TODO 항목 ID

    Returns:
        None

    Raises:
        NotFoundError: 해당 ID의 항목이 없는 경우
        ValidationError: ID 형식이 잘못된 경우
        DatabaseError: 데이터베이스 삭제 실패 시
    """
```

**Input Validation**:
- `todo_id`: 양의 정수, 존재하는 ID

**Output Contract**:
- None 반환 (성공 시)
- 삭제된 항목은 더 이상 조회 불가

## Data Transfer Objects

### TodoItem DTO

```python
@dataclass
class TodoItem:
    """TODO 항목 데이터 전송 객체"""

    id: int
    title: str
    status: str  # "incomplete" | "complete"
    priority: str  # "low" | "medium" | "high"
    due_date: Optional[str]  # "YYYY-MM-DD" | None
    created_at: str  # ISO 8601 datetime
    updated_at: str  # ISO 8601 datetime

    def is_completed(self) -> bool:
        """완료 여부 확인"""
        return self.status == "complete"

    def is_overdue(self) -> bool:
        """마감일 초과 여부 확인"""
        if not self.due_date:
            return False
        return datetime.fromisoformat(self.due_date).date() < datetime.now().date()

    def days_until_due(self) -> Optional[int]:
        """마감일까지 남은 일수 계산"""
        if not self.due_date:
            return None
        due = datetime.fromisoformat(self.due_date).date()
        today = datetime.now().date()
        return (due - today).days
```

### Filter DTO

```python
@dataclass
class TodoFilter:
    """TODO 조회 필터"""

    status: Optional[str] = None
    priority: Optional[str] = None

    def is_empty(self) -> bool:
        """필터가 비어있는지 확인"""
        return self.status is None and self.priority is None
```

## Exception Hierarchy

### Base Exceptions

```python
class TodoError(Exception):
    """기본 TODO 애플리케이션 에러"""
    pass

class ValidationError(TodoError):
    """입력 검증 에러"""
    pass

class NotFoundError(TodoError):
    """리소스 없음 에러"""
    pass

class DatabaseError(TodoError):
    """데이터베이스 작업 에러"""
    pass
```

### Exception Details

#### ValidationError
**Usage**: 입력 값 검증 실패 시
```python
raise ValidationError("제목은 필수입니다")
raise ValidationError("우선순위는 low, medium, high 중 하나여야 합니다")
```

#### NotFoundError
**Usage**: 요청한 리소스가 존재하지 않을 때
```python
raise NotFoundError(f"ID {todo_id}인 TODO 항목을 찾을 수 없습니다")
```

#### DatabaseError
**Usage**: 데이터베이스 작업 실패 시
```python
raise DatabaseError("데이터베이스 연결에 실패했습니다")
raise DatabaseError("TODO 항목 저장에 실패했습니다")
```

## Service Implementation Contract

### Constructor

```python
class TodoService:
    def __init__(self, database_url: str):
        """
        TodoService 초기화

        Args:
            database_url: 데이터베이스 연결 URL
        """
        self.database_url = database_url
        # 데이터베이스 연결 초기화
        # 테이블 존재 확인 및 생성
```

### Transaction Management

**Atomic Operations**: 각 메소드는 독립적인 트랜잭션으로 실행
- 성공 시 자동 커밋
- 실패 시 자동 롤백
- 외부 트랜잭션 컨텍스트 지원 가능

### Connection Pooling

**Strategy**: SQLAlchemy 엔진을 통한 연결 풀링
- 최대 연결 수: 5개
- 연결 타임아웃: 30초
- 자동 재연결: 예외 발생 시

## Testing Contracts

### Unit Test Contracts

#### Service Method Testing

**create_todo 테스트**:
```python
def test_create_todo_success():
    service = TodoService(":memory:")
    todo = service.create_todo("테스트 제목", "high", "2026-05-10")

    assert todo.id > 0
    assert todo.title == "테스트 제목"
    assert todo.status == "incomplete"
    assert todo.priority == "high"
    assert todo.due_date == "2026-05-10"

def test_create_todo_validation_error():
    service = TodoService(":memory:")

    with pytest.raises(ValidationError, match="제목은 필수입니다"):
        service.create_todo("")
```

#### Exception Testing

**NotFoundError 테스트**:
```python
def test_get_todo_by_id_not_found():
    service = TodoService(":memory:")

    with pytest.raises(NotFoundError, match="ID 999인 TODO 항목을 찾을 수 없습니다"):
        service.get_todo_by_id(999)
```

### Integration Test Contracts

#### Database Integration

**Full CRUD Cycle**:
```python
def test_todo_crud_cycle():
    service = TodoService(":memory:")

    # Create
    todo = service.create_todo("통합 테스트")
    assert todo.id == 1

    # Read
    retrieved = service.get_todo_by_id(1)
    assert retrieved.title == "통합 테스트"

    # Update
    updated = service.update_todo_status(1, "complete")
    assert updated.status == "complete"

    # Delete
    service.delete_todo(1)

    # Verify deletion
    with pytest.raises(NotFoundError):
        service.get_todo_by_id(1)
```

#### Transaction Rollback

**Error Recovery Test**:
```python
def test_transaction_rollback_on_error():
    service = TodoService(":memory:")

    # 정상 생성
    todo1 = service.create_todo("정상 항목")

    # 에러 발생 시 롤백 검증
    with pytest.raises(DatabaseError):
        # 강제 에러 발생 시뮬레이션
        service._simulate_db_error()

    # 기존 데이터 유지 확인
    retrieved = service.get_todo_by_id(todo1.id)
    assert retrieved.title == "정상 항목"
```

## Performance Contracts

### Response Time Guarantees

| Operation | Max Time | Conditions |
|-----------|----------|------------|
| create_todo | 50ms | 정상 입력 |
| get_todos | 100ms | 1000개 항목 |
| get_todo_by_id | 10ms | 존재하는 ID |
| update_todo_status | 20ms | 정상 업데이트 |
| delete_todo | 20ms | 존재하는 ID |

### Scalability Targets

- **Concurrent Users**: 1명 (싱글 유저 앱)
- **Max Items**: 10,000개 항목
- **Storage**: ~12MB (10,000개 항목 기준)
- **Memory**: ~25MB (1000개 항목 로드 시)

## Implementation Notes

### Dependency Injection

**Service Factory Pattern**:
```python
def create_todo_service(database_url: str) -> TodoService:
    """TodoService 팩토리 함수"""
    return TodoService(database_url)
```

### Logging Integration

**Structured Logging**:
```python
import logging

logger = logging.getLogger(__name__)

class TodoService:
    def create_todo(self, title: str, ...) -> TodoItem:
        logger.info("Creating todo", extra={"title": title, "priority": priority})
        try:
            # 구현
            logger.info("Todo created successfully", extra={"todo_id": todo.id})
            return todo
        except Exception as e:
            logger.error("Failed to create todo", extra={"error": str(e)}, exc_info=True)
            raise
```

### Configuration Management

**Environment Variables**:
- `TODO_DATABASE_URL`: 데이터베이스 연결 URL (기본: `~/.todo/todos.db`)
- `TODO_LOG_LEVEL`: 로깅 레벨 (기본: INFO)

## Conclusion

정의된 서비스 인터페이스는 레이어 분리와 테스트 용이성을 보장한다.

**Key Benefits**:
- ✅ **모듈성**: CLI와 데이터 액세스 계층 분리
- ✅ **테스트성**: 명확한 인터페이스로 단위 테스트 용이
- ✅ **유지보수성**: 인터페이스 변경 시 영향 범위 제한
- ✅ **확장성**: 새로운 기능 추가 시 인터페이스 확장 가능

**Ready for Implementation**: Phase 2 구현으로 진행 가능

---

**Interface Version**: 1.0 | **Defined**: 2026-05-03 | **Interface Designer**: Service Architecture Team