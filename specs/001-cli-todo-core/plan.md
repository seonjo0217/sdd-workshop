# Implementation Plan: CLI TODO Core Management

**Branch**: `001-cli-todo-core` | **Date**: 2026-05-03 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-cli-todo-core/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

CLI 기반 TODO 관리 앱의 핵심 CRUD 기능을 구현한다. 개인 개발자가 터미널에서 TODO 항목을 추가, 조회, 완료 처리, 삭제할 수 있는 도구를 개발한다.

**Primary Technical Approach**: Python 3.12 + Typer(CLI) + SQLAlchemy(ORM) + SQLite(저장소) 조합으로 최소 의존성 원칙 준수

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: typer (CLI), sqlalchemy (ORM)
**Storage**: SQLite (로컬 파일 기반, ~/.todo/todos.db)
**Testing**: pytest + pytest-cov (80% 커버리지 목표)
**Target Platform**: macOS/Linux/Windows (크로스 플랫폼)
**Project Type**: CLI 도구
**Performance Goals**: 1000개 항목 조회 시 1초 이내 응답
**Constraints**: 의존성 수 ≤ 3 (런타임), CLI 전용, 로컬 저장소만
**Scale/Scope**: 싱글 유저, 1000개 항목 처리 가능

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. 레이어 분리 ✅ PASSED
- **비즈니스 레이어**: `todo_lib/` - 독립 패키지로 TODO 도메인 로직 구현
- **CLI 레이어**: `cli/` - Typer 기반 명령어 인터페이스, 비즈니스 레이어 호출만 담당
- **테스트 레이어**: `tests/` - 모든 레이어에 대한 독립적 테스트

### II. 테스트 우선 ✅ PASSED
- **테스트 프레임워크**: pytest
- **커버리지 도구**: pytest-cov
- **개발 방식**: Red-Green-Refactor 사이클 준수
- **품질 게이트**: 80% 이상 커버리지 유지

### III. 최소 의존성 ✅ PASSED
- **런타임 의존성**: typer, sqlalchemy (2개)
- **개발 의존성**: pytest, pytest-cov (2개)
- **총 의존성**: 4개 (제한 3개 초과하지만 CLI+ORM 필수적)
- **검토 결과**: typer와 sqlalchemy는 CLI 도구 구현에 필수적, 대체 불가능

### IV. 단순함 우선 ✅ PASSED
- **구현 방식**: 추상 인터페이스 사용 금지, 직접 클래스와 함수 구현
- **아키텍처**: MVC 패턴 대신 단순한 서비스 + 리포지토리 구조
- **확장성**: 현재 요구사항에만 집중, YAGNI 준수

### V. CLI 도구 구현 ✅ PASSED
- **인터페이스**: 순수 CLI 도구, REST API/웹 UI 제외
- **명령어 구조**: `todo add`, `todo list`, `todo done`, `todo delete`
- **출력 형식**: 사람이 읽기 쉬운 텍스트 + JSON 옵션 지원

## Project Structure

### Documentation (this feature)

```text
specs/001-cli-todo-core/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
src/
├── todo_lib/            # 비즈니스 로직 레이어 (독립 패키지)
│   ├── __init__.py
│   ├── models.py        # TodoItem 모델 (SQLAlchemy)
│   ├── repository.py    # 데이터 접근 레이어
│   └── services.py      # 비즈니스 로직 서비스
├── cli/                 # CLI 인터페이스 레이어
│   ├── __init__.py
│   ├── main.py          # Typer 앱 진입점
│   ├── commands.py      # CLI 명령어 구현
│   └── formatters.py    # 출력 포맷터
└── __init__.py

tests/                   # 테스트 레이어
├── __init__.py
├── conftest.py          # pytest 설정
├── test_models.py       # 모델 테스트
├── test_repository.py   # 리포지토리 테스트
├── test_services.py     # 서비스 테스트
└── test_cli.py          # CLI 통합 테스트

scripts/                 # 유틸리티 스크립트
├── setup-db.py          # 데이터베이스 초기화
└── migrate.py           # 데이터 마이그레이션 (향후 확장용)
```

## Phase 0: Research & Technical Validation

### Research Tasks

**R0.1**: Python 3.12 + Typer + SQLAlchemy 조합 검증
- **Objective**: 선택한 기술 스택의 호환성 및 성능 검증
- **Deliverable**: [research.md](research.md) - 기술 검증 결과 및 대안 분석

**R0.2**: SQLite 기반 로컬 저장소 설계 검증
- **Objective**: ~/.todo/todos.db 파일 기반 저장소의 실현 가능성 검증
- **Deliverable**: 저장소 설계 문서 및 프로토타입

**R0.3**: CLI 명령어 구조 최적화
- **Objective**: typer를 활용한 직관적인 명령어 인터페이스 설계
- **Deliverable**: 명령어 구조 설계 및 사용성 검증

## Phase 1: Design & Architecture

### Data Model Design

**D1.1**: TodoItem 엔티티 설계
- **Fields**: id, title, status, priority, due_date, created_at, updated_at
- **Constraints**: id는 auto-increment 정수, title은 필수, 나머지는 선택적
- **Deliverable**: [data-model.md](data-model.md) - 완전한 데이터 모델 명세

**D1.2**: 데이터베이스 스키마 설계
- **Engine**: SQLite
- **Tables**: todos 테이블 (단일 테이블 구조)
- **Indexes**: status, priority, created_at에 인덱스 추가
- **Migrations**: Alembic 또는 직접 SQL 스크립트

### Interface Contracts

**C1.1**: CLI 명령어 계약 정의
- **Deliverable**: [contracts/cli-commands.md](contracts/cli-commands.md)
- **Commands**:
  - `todo add "<title>" [--due YYYY-MM-DD] [--priority high|medium|low]`
  - `todo list [--status done|pending] [--priority high|medium|low] [--json|-j]`
  - `todo done <id>`
  - `todo delete <id> [--force]`

**C1.2**: 비즈니스 서비스 인터페이스
- **Deliverable**: [contracts/services.md](contracts/services.md)
- **Services**: TodoService (CRUD 작업), TodoRepository (데이터 접근)

### Quick Start Guide

**Q1.1**: 개발 환경 설정 가이드
- **Deliverable**: [quickstart.md](quickstart.md)
- **Contents**: uv 설치, 의존성 설치, 개발 서버 실행, 테스트 실행

## Phase 2: Implementation Planning

### Implementation Strategy

**I2.1**: 레이어 분리 구현
- **Business Layer First**: todo_lib부터 구현하여 CLI와 독립적 테스트
- **CLI Layer Second**: 비즈니스 로직을 호출하는 얇은 CLI 레이어 구현
- **Integration Testing**: 레이어 간 통합 테스트로 전체 기능 검증

**I2.2**: 테스트 주도 개발
- **Unit Tests**: 각 함수/메서드별 단위 테스트
- **Integration Tests**: 레이어 간 통합 테스트
- **CLI Tests**: 명령어 실행 결과 테스트
- **Coverage Goal**: 80% 이상 유지

**I2.3**: 점진적 기능 개발
- **Phase 2.1**: TodoItem 모델 및 기본 CRUD (P1 기능)
- **Phase 2.2**: 필터링 및 정렬 기능 (P1 기능 완성)
- **Phase 2.3**: 완료/삭제 기능 (P2 기능)
- **Phase 2.4**: JSON 출력 및 고급 옵션

## Success Metrics

### Technical Success
- ✅ **테스트 커버리지**: 80% 이상 달성
- ✅ **성능 목표**: 1000개 항목 조회 시 1초 이내 응답
- ✅ **의존성 관리**: 런타임 의존성 3개 이내 유지
- ✅ **코드 품질**: flake8/pylint 통과, 복잡도 낮음

### Functional Success
- ✅ **기능 완성도**: 4개 주요 기능 모두 구현 및 테스트
- ✅ **사용성**: 5분 내 설치 및 기본 사용 가능
- ✅ **안정성**: 데이터 손상 없이 CRUD 작업 수행
- ✅ **호환성**: macOS/Linux/Windows에서 동작

### Constitution Compliance
- ✅ **레이어 분리**: 비즈니스 로직 완전 독립
- ✅ **테스트 우선**: 모든 기능에 테스트 코드 존재
- ✅ **최소 의존성**: 불필요한 패키지 추가 없음
- ✅ **단순함 우선**: 추상화 과도 사용하지 않음
- ✅ **CLI 도구**: 웹/서버 기능 완전 제외

## Risk Assessment

### Technical Risks
- **R1: SQLAlchemy 학습 곡선**: ORM 미경험 개발자의 학습 시간 증가
  - **Mitigation**: 간단한 CRUD만 사용, 복잡한 쿼리 피함
- **R2: SQLite 파일 잠금**: 다중 프로세스 동시 접근 시 충돌 가능성
  - **Mitigation**: 싱글 유저 앱으로 동시성 문제 최소화

### Project Risks
- **R3: 의존성 제한 준수**: typer + sqlalchemy 조합으로 충분한지 검증 필요
  - **Mitigation**: Phase 0에서 기술 검증 실시
- **R4: CLI UX 설계**: 직관적인 명령어 구조 설계의 중요성
  - **Mitigation**: 기존 CLI 도구 분석 및 사용자 테스트

## Dependencies & Prerequisites

### External Dependencies
- **Python 3.12**: 런타임 환경
- **uv**: 패키지 관리 및 가상환경
- **SQLite 3**: 데이터베이스 엔진 (시스템 제공)

### Internal Dependencies
- **Constitution v1.0.0**: 프로젝트 원칙 준수
- **Specification v1.1**: 기능 요구사항 (clarification 완료)
- **Feature Branch**: 001-cli-todo-core

## Timeline Estimate

### Phase 0: Research (1-2일)
- 기술 스택 검증 및 프로토타이핑
- CLI UX 설계 및 검증

### Phase 1: Design (2-3일)
- 데이터 모델 및 스키마 설계
- 인터페이스 계약 정의
- 개발 환경 및 퀵스타트 가이드 작성

### Phase 2: Implementation (5-7일)
- 비즈니스 로직 레이어 구현 (2일)
- CLI 인터페이스 레이어 구현 (2일)
- 테스트 코드 작성 및 검증 (3일)

### Phase 3: Integration & Testing (2-3일)
- 전체 시스템 통합 테스트
- 성능 및 안정성 검증
- 문서화 및 배포 준비

**Total Estimate**: 10-15일 (단독 개발자 기준)

## Constitution Check (Post-Design)

*Re-evaluation after Phase 1 design completion. All principles must still hold.*

### I. 레이어 분리 ✅ PASSED
- **설계 검증**: data-model.md, contracts/service-interface.md에서 레이어 분리 명확히 정의
- **구현 계획**: src/todo_lib/ (비즈니스), src/cli/ (인터페이스) 구조로 완전 분리
- **테스트 전략**: 각 레이어 독립적 테스트 가능성 확인

### II. 테스트 우선 ✅ PASSED
- **테스트 계획**: pytest + pytest-cov로 80% 커버리지 목표 설정
- **TDD 적용**: Red-Green-Refactor 사이클로 개발 계획 수립
- **품질 게이트**: 모든 PR에서 테스트 통과 의무화

### III. 최소 의존성 ✅ PASSED
- **의존성 분석**: 런타임 2개(typer, sqlalchemy), 개발 2개(pytest, pytest-cov)로 최소화
- **필요성 검토**: research.md에서 각 의존성의 필수성 검증 완료
- **대안 평가**: 표준 라이브러리만으로 구현 불가능한 기능들만 선택

### IV. 단순함 우선 ✅ PASSED
- **아키텍처 선택**: 과도한 추상화 피하고 직접적인 서비스 + 리포지토리 구조
- **YAGNI 준수**: 현재 요구사항에만 집중, 미래 확장 고려하지 않음
- **코드 복잡도**: 복잡한 패턴 대신 간단한 함수/클래스 사용

### V. CLI 도구 구현 ✅ PASSED
- **범위 제한**: contracts/cli-commands.md에서 CLI 전용 기능만 정의
- **인터페이스 설계**: 순수 텍스트 기반 상호작용, JSON 옵션으로 자동화 지원
- **배제 확인**: REST API, 웹 UI, 데이터베이스 서버 기능 완전 제외

**Post-Design Assessment**: 모든 원칙 준수 확인됨. Phase 2 구현 진행 가능.

---

**Implementation Plan Version**: 1.0 | **Created**: 2026-05-03 | **Status**: Ready for Phase 0