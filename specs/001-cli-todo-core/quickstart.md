# Quick Start Guide

**Feature**: CLI TODO Core Management | **Date**: 2026-05-03
**Phase**: 1 (Design & Architecture)

## Overview

CLI TODO 앱의 개발 환경 설정과 기본 사용법을 안내한다. Python 3.12 + uv 패키지 매니저를 사용한 빠른 시작 가이드이다.

**Prerequisites**:
- Python 3.12 이상
- uv 패키지 매니저
- Git (선택사항)

## Development Environment Setup

### 1. Python 3.12 설치 확인

```bash
# Python 버전 확인
python --version
# Python 3.12.x 이상이어야 함

# 또는 python3 명령어 사용
python3 --version
```

**macOS 설치 (Homebrew 사용)**:
```bash
brew install python@3.12
```

**Ubuntu/Debian 설치**:
```bash
# 데드스냅 설치 권장
sudo apt update
sudo apt install python3.12 python3.12-venv
```

### 2. uv 패키지 매니저 설치

```bash
# 공식 설치 스크립트
curl -LsSf https://astral.sh/uv/install.sh | sh

# 설치 확인
uv --version
# uv 0.x.x
```

### 3. 프로젝트 초기화

```bash
# 프로젝트 디렉토리 생성
mkdir cli-todo-app
cd cli-todo-app

# Git 저장소 초기화 (선택사항)
git init

# Python 프로젝트 초기화
uv init

# 의존성 추가
uv add typer sqlalchemy pytest pytest-cov

# 개발 의존성 추가
uv add --dev pytest pytest-cov
```

### 4. 프로젝트 구조 생성

```bash
# 기본 디렉토리 구조 생성
mkdir -p src/todo_lib src/cli tests

# __init__.py 파일 생성
touch src/__init__.py
touch src/todo_lib/__init__.py
touch src/cli/__init__.py
touch tests/__init__.py
```

### 5. pyproject.toml 설정

```toml
[project]
name = "cli-todo"
version = "0.1.0"
description = "CLI 기반 TODO 관리 앱"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "typer>=0.12.0",
    "sqlalchemy>=2.0.0",
]

[project.scripts]
todo = "cli.main:app"

[tool.uv]
dev-dependencies = [
    "pytest>=8.0.0",
    "pytest-cov>=5.0.0",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = "-v --cov=src --cov-report=term-missing"
```

## Basic Usage Examples

### 첫 번째 TODO 추가

```bash
# 기본 우선순위로 TODO 추가
todo add "프로젝트 계획서 작성"

# 마감일과 우선순위 지정
todo add "코드 리뷰 완료" --due 2026-05-10 --priority high

# 긴 제목의 TODO 추가
todo add "데이터베이스 스키마 설계 및 마이그레이션 스크립트 작성"
```

### TODO 목록 조회

```bash
# 모든 TODO 조회
todo list

# 완료된 항목만 조회
todo list --status done

# 미완료 고우선순위 항목 조회
todo list --status pending --priority high

# JSON 형식으로 출력
todo list --json
```

### TODO 완료 처리

```bash
# ID로 완료 처리
todo done 1

# 여러 항목 완료 (반복 실행)
todo done 2
todo done 3
```

### TODO 삭제

```bash
# 확인 후 삭제
todo delete 1
# "정말로 삭제하시겠습니까? (y/N):" 프롬프트 표시

# 확인 없이 강제 삭제
todo delete 2 --force
```

## Development Workflow

### 1. 코드 작성

**비즈니스 로직 (src/todo_lib/)**:
```python
# src/todo_lib/models.py
from dataclasses import dataclass
from typing import Optional

@dataclass
class TodoItem:
    id: int
    title: str
    status: str
    priority: str
    due_date: Optional[str]
    created_at: str
    updated_at: str
```

**서비스 계층 (src/todo_lib/)**:
```python
# src/todo_lib/service.py
from sqlalchemy import create_engine, text
from .models import TodoItem

class TodoService:
    def __init__(self, database_url: str = "~/.todo/todos.db"):
        self.engine = create_engine(f"sqlite:///{database_url}")

    def create_todo(self, title: str, priority: str = "medium", due_date: Optional[str] = None) -> TodoItem:
        # 구현
        pass
```

**CLI 인터페이스 (src/cli/)**:
```python
# src/cli/main.py
import typer
from todo_lib.service import TodoService

app = typer.Typer()
service = TodoService()

@app.command()
def add(
    title: str = typer.Argument(..., help="TODO 제목"),
    due: Optional[str] = typer.Option(None, help="마감일"),
    priority: str = typer.Option("medium", help="우선순위")
):
    """새 TODO 항목 추가"""
    todo = service.create_todo(title, priority, due)
    typer.echo(f"✅ Added: {todo.title} (ID: {todo.id})")
```

### 2. 테스트 실행

```bash
# 모든 테스트 실행
uv run pytest

# 코드 커버리지 포함
uv run pytest --cov

# 특정 테스트 파일 실행
uv run pytest tests/test_service.py

# 상세 출력
uv run pytest -v
```

### 3. 패키지 빌드 및 설치

```bash
# 개발 모드로 설치
uv pip install -e .

# 또는 빌드 후 설치
uv build
uv pip install dist/*.whl
```

### 4. 실행 및 테스트

```bash
# CLI 명령어 실행
todo --help

# 기본 기능 테스트
todo add "Hello World"
todo list
todo done 1
todo list
```

## Testing Strategy

### 단위 테스트 작성

```python
# tests/test_service.py
import pytest
from todo_lib.service import TodoService

class TestTodoService:
    def setup_method(self):
        self.service = TodoService(":memory:")

    def test_create_todo_success(self):
        todo = self.service.create_todo("테스트")
        assert todo.title == "테스트"
        assert todo.status == "incomplete"

    def test_create_todo_validation_error(self):
        with pytest.raises(ValueError):
            self.service.create_todo("")
```

### 통합 테스트 작성

```python
# tests/test_cli.py
import subprocess
import json

def test_cli_add_and_list():
    # TODO 추가
    result = subprocess.run(
        ["todo", "add", "통합 테스트"],
        capture_output=True, text=True
    )
    assert result.returncode == 0
    assert "Added" in result.stdout

    # 목록 조회
    result = subprocess.run(
        ["todo", "list", "--json"],
        capture_output=True, text=True
    )
    assert result.returncode == 0

    data = json.loads(result.stdout)
    assert len(data["todos"]) == 1
    assert data["todos"][0]["title"] == "통합 테스트"
```

## Troubleshooting

### 일반적인 문제 해결

**1. Python 버전 문제**
```bash
# pyenv를 사용한 버전 관리
pyenv install 3.12.0
pyenv local 3.12.0
python --version  # 3.12.0 확인
```

**2. uv 설치 문제**
```bash
# 수동 설치
curl -L https://github.com/astral-sh/uv/releases/latest/download/uv-installer.sh | sh
```

**3. 패키지 설치 실패**
```bash
# 캐시 정리 후 재시도
uv cache clean
uv sync
```

**4. 데이터베이스 파일 권한 문제**
```bash
# 데이터베이스 디렉토리 생성
mkdir -p ~/.todo
chmod 755 ~/.todo
```

### 디버깅 팁

**CLI 디버깅**:
```bash
# 상세 로깅 활성화
export TODO_LOG_LEVEL=DEBUG
todo list

# Python 디버거 사용
python -m pdb -c "import cli.main; cli.main.app()"
```

**데이터베이스 검사**:
```bash
# SQLite 직접 확인
sqlite3 ~/.todo/todos.db
.schema todos
SELECT * FROM todos;
.quit
```

## Performance Tuning

### 개발 환경 최적화

**1. uv 캐시 활용**:
```bash
# 캐시 위치 확인
uv cache dir

# 캐시 정리 (문제 발생 시)
uv cache clean
```

**2. pytest 최적화**:
```bash
# 병렬 테스트 실행
uv run pytest -n auto

# 변경된 파일만 테스트
uv run pytest --lf
```

### 프로덕션 배포

**1. 의존성 잠금**:
```bash
# uv.lock 파일 생성
uv lock

# 잠금 파일 사용 설치
uv sync --locked
```

**2. 실행 파일 생성**:
```bash
# PyInstaller로 독립 실행 파일 생성
uv add pyinstaller
uv run pyinstaller --onefile src/cli/main.py --name todo

# 생성된 실행 파일 사용
./dist/todo --help
```

## Next Steps

### Phase 2 구현 시작

1. **데이터 모델 구현**: `src/todo_lib/models.py`
2. **서비스 계층 구현**: `src/todo_lib/service.py`
3. **CLI 인터페이스 구현**: `src/cli/main.py`
4. **테스트 코드 작성**: `tests/`
5. **통합 및 배포**

### 추가 리소스

- **공식 문서**:
  - [Typer Documentation](https://typer.tiangolo.com/)
  - [SQLAlchemy Documentation](https://sqlalchemy.org/)
  - [uv Documentation](https://docs.astral.sh/uv/)

- **커뮤니티**:
  - [Python Discord](https://discord.gg/python)
  - [FastAPI/Typer Discord](https://discord.gg/VQjSZaeJmf)

## Conclusion

이 가이드를 따라 개발 환경을 설정하고 기본적인 TODO 앱을 구현할 수 있다.

**Key Takeaways**:
- ✅ **빠른 시작**: uv로 5분 내 개발 환경 구축
- ✅ **표준 도구**: Python 3.12 + Typer + SQLAlchemy
- ✅ **테스트 우선**: pytest로 품질 보장
- ✅ **실행 가능**: 즉시 사용 가능한 CLI 앱

**Ready for Development**: Phase 2 구현 시작 가능

---

**Guide Version**: 1.0 | **Created**: 2026-05-03 | **Guide Author**: Development Setup Team