# CLI Command Contracts

**Feature**: CLI TODO Core Management | **Date**: 2026-05-03
**Phase**: 1 (Design & Architecture)

## Overview

CLI 명령어 인터페이스의 계약을 정의한다. 각 명령어의 입력, 출력, 에러 처리를 명세하여 일관된 사용자 경험을 보장한다.

**Contract Principles**:
- **일관성**: 모든 명령어가 동일한 패턴 준수
- **안전성**: 파괴적 작업에 확인 메커니즘
- **유연성**: 옵션 파라미터로 다양한 사용 사례 지원
- **명확성**: 직관적인 명령어 이름과 도움말

## Command Structure

### Base Command: `todo`

```bash
todo [COMMAND] [OPTIONS] [ARGS]
```

**Available Commands**:
- `add` - 새 TODO 항목 추가
- `list` - TODO 항목 목록 조회
- `done` - TODO 항목 완료 처리
- `delete` - TODO 항목 삭제

## Command Contracts

### 1. Add Command

#### Signature
```bash
todo add <title> [--due DATE] [--priority PRIORITY]
```

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `title` | string | ✅ | - | TODO 항목 제목 (1-256자) |
| `--due` | date | ❌ | null | 마감일 (YYYY-MM-DD 형식) |
| `--priority` | enum | ❌ | medium | 우선순위 (low, medium, high) |

#### Input Validation

**Title Validation**:
- 필수 입력
- 공백 제거 후 1-256자 범위
- 빈 문자열 불허용

**Due Date Validation**:
- 선택적 입력
- YYYY-MM-DD 형식
- 유효한 날짜 검증

**Priority Validation**:
- 선택적 입력
- 허용 값: low, medium, high
- 기본값: medium

#### Success Output

**Format**: Human-readable text
```
✅ Added TODO: "프로젝트 계획서 작성" (ID: 1)
   Due: 2026-05-10, Priority: high
```

**JSON Format** (when requested):
```json
{
  "success": true,
  "action": "add",
  "todo": {
    "id": 1,
    "title": "프로젝트 계획서 작성",
    "status": "incomplete",
    "priority": "high",
    "due_date": "2026-05-10",
    "created_at": "2026-05-03T09:00:00Z",
    "updated_at": "2026-05-03T09:00:00Z"
  }
}
```

#### Error Cases

**E001: Invalid Title**
```
❌ Error: 제목은 필수입니다
Usage: todo add <title> [--due DATE] [--priority PRIORITY]
```

**E002: Title Too Long**
```
❌ Error: 제목은 256자를 초과할 수 없습니다
```

**E003: Invalid Date Format**
```
❌ Error: 마감일은 YYYY-MM-DD 형식이어야 합니다
```

**E004: Invalid Priority**
```
❌ Error: 우선순위는 low, medium, high 중 하나여야 합니다
```

### 2. List Command

#### Signature
```bash
todo list [--status STATUS] [--priority PRIORITY] [--json]
```

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `--status` | enum | ❌ | null | 상태 필터 (done, pending) |
| `--priority` | enum | ❌ | null | 우선순위 필터 (low, medium, high) |
| `--json` | flag | ❌ | false | JSON 형식 출력 |

#### Status Mapping
- `done` → `complete`
- `pending` → `incomplete`

#### Success Output

**Format**: Human-readable table
```
📋 TODO 목록 (2개 항목)

ID  Status     Priority  Due Date    Title
────────────────────────────────────────────────────────────
1   ⏳ pending  high      2026-05-10  프로젝트 계획서 작성
2   ✅ done     medium    -           코드 리뷰 완료

총 2개 항목 (완료: 1개, 미완료: 1개)
```

**Empty State**:
```
📋 TODO 목록 (0개 항목)

아직 등록된 TODO가 없습니다. 'todo add "제목"'으로 첫 번째 항목을 추가하세요.
```

**JSON Format**:
```json
{
  "total": 2,
  "completed": 1,
  "pending": 1,
  "todos": [
    {
      "id": 1,
      "title": "프로젝트 계획서 작성",
      "status": "incomplete",
      "priority": "high",
      "due_date": "2026-05-10",
      "created_at": "2026-05-03T09:00:00Z",
      "updated_at": "2026-05-03T09:00:00Z"
    },
    {
      "id": 2,
      "title": "코드 리뷰 완료",
      "status": "complete",
      "priority": "medium",
      "due_date": null,
      "created_at": "2026-05-03T08:00:00Z",
      "updated_at": "2026-05-03T08:30:00Z"
    }
  ]
}
```

#### Error Cases

**E101: Database Error**
```
❌ Error: 데이터베이스에 접근할 수 없습니다
```

### 3. Done Command

#### Signature
```bash
todo done <id>
```

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | integer | ✅ | - | 완료 처리할 TODO 항목 ID |

#### Input Validation

**ID Validation**:
- 필수 입력
- 양의 정수
- 존재하는 ID여야 함

#### Success Output

**Format**: Human-readable text
```
✅ Marked as done: "프로젝트 계획서 작성" (ID: 1)
```

**JSON Format**:
```json
{
  "success": true,
  "action": "done",
  "todo": {
    "id": 1,
    "title": "프로젝트 계획서 작성",
    "status": "complete",
    "updated_at": "2026-05-03T10:00:00Z"
  }
}
```

#### Error Cases

**E201: Invalid ID Format**
```
❌ Error: ID는 숫자여야 합니다
```

**E202: Todo Not Found**
```
❌ Error: ID 999인 TODO 항목을 찾을 수 없습니다
```

**E203: Already Completed**
```
⚠️  Warning: ID 1은 이미 완료된 항목입니다
```

### 4. Delete Command

#### Signature
```bash
todo delete <id> [--force]
```

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | integer | ✅ | - | 삭제할 TODO 항목 ID |
| `--force` | flag | ❌ | false | 확인 없이 강제 삭제 |

#### Input Validation

**ID Validation**:
- 필수 입력
- 양의 정수
- 존재하는 ID여야 함

#### Confirmation Flow (when --force not used)

**Prompt**:
```
정말로 "프로젝트 계획서 작성" (ID: 1)을 삭제하시겠습니까? (y/N):
```

**User Input Handling**:
- `y`, `Y`, `yes`, `Yes` → 삭제 진행
- `n`, `N`, `no`, `No`, `Enter` → 삭제 취소
- 기타 입력 → 재프롬프트

#### Success Output

**Format**: Human-readable text
```
🗑️  Deleted: "프로젝트 계획서 작성" (ID: 1)
```

**JSON Format**:
```json
{
  "success": true,
  "action": "delete",
  "deleted_todo": {
    "id": 1,
    "title": "프로젝트 계획서 작성"
  }
}
```

#### Error Cases

**E301: Invalid ID Format**
```
❌ Error: ID는 숫자여야 합니다
```

**E302: Todo Not Found**
```
❌ Error: ID 999인 TODO 항목을 찾을 수 없습니다
```

**E303: Deletion Cancelled**
```
❌ 삭제가 취소되었습니다
```

## Global Options

### Help Option

**Signature**: `--help`, `-h`

**Output**: 각 명령어별 상세 도움말
```
Usage: todo [COMMAND] [OPTIONS]

CLI TODO 관리 도구

Commands:
  add     새 TODO 항목 추가
  list    TODO 항목 목록 조회
  done    TODO 항목 완료 처리
  delete  TODO 항목 삭제

Options:
  --help, -h  도움말 표시
```

### Version Option

**Signature**: `--version`, `-v`

**Output**:
```
todo v1.0.0
```

## Error Handling

### Exit Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 0 | Success | 명령어 성공적 실행 |
| 1 | General Error | 일반적인 오류 |
| 2 | Validation Error | 입력 값 검증 실패 |
| 3 | Not Found | 리소스를 찾을 수 없음 |
| 4 | Database Error | 데이터베이스 관련 오류 |

### Error Message Format

**Standard Format**:
```
❌ Error: [에러 메시지]
[추가 컨텍스트 또는 사용법]
```

**Examples**:
```
❌ Error: 제목은 필수입니다
Usage: todo add <title> [--due DATE] [--priority PRIORITY]
```

## Testing Contracts

### Command Line Testing

**Add Command Test**:
```bash
# 성공 케이스
$ todo add "프로젝트 계획서 작성" --due 2026-05-10 --priority high
✅ Added TODO: "프로젝트 계획서 작성" (ID: 1)

# 에러 케이스
$ todo add ""
❌ Error: 제목은 필수입니다
```

**List Command Test**:
```bash
# 기본 목록
$ todo list
📋 TODO 목록 (1개 항목)
...

# 필터링
$ todo list --status pending --priority high
📋 TODO 목록 (1개 항목)
...
```

### JSON Output Testing

**JSON Validation**:
```bash
$ todo list --json | jq '.total'
1
```

## Implementation Notes

### Typer Integration

**Command Definition Pattern**:
```python
import typer

app = typer.Typer()

@app.command()
def add(
    title: str = typer.Argument(..., help="TODO 항목 제목"),
    due: str = typer.Option(None, help="마감일 (YYYY-MM-DD)"),
    priority: str = typer.Option("medium", help="우선순위")
):
    """새 TODO 항목 추가"""
    # 구현 로직
    pass
```

### Error Handling Pattern

**Custom Exception Classes**:
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
```

### Output Formatting

**Rich Library Integration**:
```python
from rich.console import Console
from rich.table import Table

console = Console()

def print_todo_list(todos):
    table = Table("ID", "Status", "Priority", "Due Date", "Title")
    for todo in todos:
        table.add_row(
            str(todo.id),
            "✅ done" if todo.status == "complete" else "⏳ pending",
            todo.priority,
            todo.due_date or "-",
            todo.title
        )
    console.print(table)
```

## Conclusion

정의된 CLI 계약은 일관성 있고 사용하기 쉬운 인터페이스를 제공한다.

**Key Features**:
- ✅ **직관성**: 동사 기반 명령어 구조
- ✅ **안전성**: 삭제 시 확인 메커니즘
- ✅ **유연성**: 필터링 및 JSON 출력 옵션
- ✅ **명확성**: 상세한 에러 메시지와 도움말

**Ready for Implementation**: Phase 2 구현으로 진행 가능

---

**Contract Version**: 1.0 | **Defined**: 2026-05-03 | **Contract Designer**: CLI Interface Team