# 구현 진행 가이드

## 1. 목적

본 문서는 현재 README에 정리된 요구사항을 기준으로, 실제 프로젝트를 구현하기 위한 **실행 순서**, **작성 방식**, **테스트 전략**을 정리한 문서이다.

## 2. 먼저 해야 할 것

### 2.1 현재 README 기준으로 구현 범위를 확정
1. 핵심 기능 우선순위를 정한다.
   - 1순위: `add`, `list`, `search`, `summary`, `delete`, `update`
   - 2순위: `budget`, `category`, `import/export`, `backup`, `recurring`
2. 구현하는 동안 README와 용어를 일치시킨다.
   - `Repository` / `Validator` / `ErrorHandler` 용어를 코드와 문서에서 동일하게 유지
3. 가장 먼저 구현할 데이터 저장 포맷을 고정한다.
   - `transactions.jsonl`
   - `categories.jsonl`
   - `budgets.jsonl`

### 2.2 프로젝트 구조를 먼저 만든다
1. `app/cli`, `app/services`, `app/repositories`, `app/models`, `app/validators`, `app/errors`, `app/utils` 디렉터리를 생성한다.
2. `main.py`를 만들고 CLI 진입점을 연결한다.
3. `data/`, `backups/`, `tests/` 디렉터리를 초기화한다.

### 2.3 공통 규칙을 먼저 정한다
1. 입력 검증은 **서비스 호출 이전**에 수행한다.
2. 저장은 **임시 파일 + 원자적 교체** 방식으로 처리한다.
3. 오류 메시지는 `오류: ...` / `해결: ...` 형식으로 통일한다.
4. 외부 라이브러리는 사용하지 않는다.

## 3. 구현 순서 추천

### 1단계: 기본 골격 만들기
- `main.py`
- `app/cli/parser.py`
- `app/cli/handlers.py`
- `app/cli/renderer.py`
- `app/models/transaction.py`
- `app/repositories/jsonl_repository.py`
- `app/repositories/transaction_repository.py`
- `app/validators/input_validator.py`
- `app/errors/app_error.py`
- `app/errors/error_handler.py`

### 2단계: 거래 CRUD 구현
- 거래 생성
- 거래 목록 출력
- 거래 검색
- 거래 수정
- 거래 삭제

### 3단계: 카테고리 및 예산 구현
- 카테고리 추가 / 목록 / 삭제
- 예산 설정 및 조회
- summary에서 예산 사용률 표시

### 4단계: CSV import/export 구현
- CSV 읽기
- CSV 쓰기
- 필터링 기반 export

### 5단계: 백업 및 반복 기능 구현
- 백업 생성
- 반복 규칙 등록
- 반복 규칙 실행

## 4. 작성 방식

### 4.1 코드 작성 원칙
- 작은 단위부터 구현하고, 완료될 때마다 실행 검증을 한다.
- 한 파일에 모든 로직을 몰아넣지 않는다.
- 각 계층의 책임을 분명히 나눈다.
- `Repository`는 파일 I/O만 담당한다.
- `Service`는 검증과 저장소 조합을 담당한다.
- `CLI`는 입력/출력과 라우팅만 담당한다.

### 4.2 구현 순서에 맞춘 작성 방식
1. 데이터 모델부터 먼저 작성한다.
2. 저장소 레이어를 구현한 뒤 서비스 레이어를 붙인다.
3. CLI는 마지막에 붙여서 흐름을 연결한다.
4. 테스트는 기능 단위로 먼저 작성한 뒤 구현을 진행한다.

## 5. 테스트 전략

### 5.1 테스트 종류
#### 단위 테스트
- 모델 구조 검증
- 검증 함수 검증
- Repository의 파일 읽기/쓰기 검증
- Service의 비즈니스 규칙 검증

#### 통합 테스트
- CLI 명령어 흐름 검증
- 저장소와 서비스 조합 검증
- import/export 흐름 검증
- 예산/요약 흐름 검증

### 5.2 테스트 작성 방식
- 기능을 구현하기 전에 먼저 테스트를 만든다.
- 가장 작은 동작부터 테스트한다.
  - 예: `TransactionRepository.append()`가 파일에 한 줄 추가되는지
  - 예: `TransactionService.add_transaction()`이 검증 실패 시 예외를 반환하는지
  - 예: `search`가 날짜/카테고리 조건을 올바르게 필터링하는지
- 테스트 데이터는 `tmp_path` 또는 테스트 전용 샘플 파일을 사용한다.

### 5.3 테스트 항목 예시
- `add` 입력 검증
- `list` 최신순 정렬
- `search` 기간/카테고리/타입 조건 필터링
- `summary` 월별 집계 및 데이터 없음 처리
- `update/delete` 존재하지 않는 ID 처리
- `category` 삭제 보호
- `import/export` CSV 파싱 및 저장
- `backup` 생성 파일 경로 검증

## 6. 테스트 실행 방식

### 6.1 기본 실행 순서
1. `pytest`로 전체 테스트를 먼저 실행한다.
2. 실패한 테스트를 기능 단위로 좁힌다.
3. 수정 후 반복 실행한다.

### 6.2 추천 명령 예시
- `pytest`
- `pytest tests/test_services.py`
- `pytest tests/test_repositories.py`
- `pytest tests/test_validators.py`
- `pytest tests/test_cli.py`

### 6.3 기대하는 결과
- 전체 테스트가 통과해야 구현 완료로 본다.
- 실패 항목이 있으면 README의 체크리스트를 업데이트한다.

## 7. 오류 처리와 검증 정책

### 7.1 구현 기준
- 예외 메시지는 사용자 친화적으로 출력한다.
- 스택트레이스를 직접 출력하지 않는다.
- CLI는 사용자 메시지 출력만 담당하고, 예외 변환은 `ErrorHandler`가 담당한다.

### 7.2 권장 정책
- 입력 오류: `2` 코드
- 일반 사용자/데이터/파일 오류: `1` 코드
- 데이터 손상 또는 내부 시스템 오류: `3` 코드

### 7.3 예외 처리 흐름
1. 서비스 또는 저장소에서 예외를 발생시킨다.
2. `ErrorHandler`가 예외를 사용자 친화 메시지로 변환한다.
3. `CLI`는 변환된 메시지를 출력한다.

## 8. 실제 진행 체크리스트

### 시작 전
- [ ] README와 구현 범위가 일치하는가
- [ ] 파일 구조가 생성되었는가
- [ ] 테스트 디렉터리가 준비되었는가
- [ ] `Repository`, `Validator`, `ErrorHandler` 용어가 코드와 문서에서 일치하는가
- [ ] 저장 파일 경로(`data/`, `backups/`)가 고정되었는가

### 1단계 완료 기준: 기본 골격
- [ ] `main.py`가 `parser.py`, `handlers.py`를 초기화하는가
- [ ] `parser.py`가 명령어와 옵션을 정의하는가
- [ ] `handlers.py`가 서비스 계층만 호출하는가
- [ ] `renderer.py`가 출력만 담당하는가
- [ ] `transaction.py` 모델이 `dataclass` 또는 동등한 구조로 정의되었는가
- [ ] `jsonl_repository.py`가 공용 JSONL 읽기/쓰기 유틸을 제공하는가
- [ ] `transaction_repository.py`가 `append`, `load_all`, `save_all`, `update`, `delete`, `filter`를 지원하는가
- [ ] `input_validator.py`가 공통 검증 인터페이스를 제공하는가
- [ ] `app_error.py`가 예외 유형을 표현하는가
- [ ] `error_handler.py`가 사용자 친화 메시지와 해결 힌트를 제공하는가

### 2단계 완료 기준: 거래 CRUD
- [ ] `add`가 대화형 입력 또는 옵션 입력을 받아 거래를 생성하는가
- [ ] `add`가 카테고리 존재 여부를 검증하는가
- [ ] `add`가 생성된 거래 ID를 출력하는가
- [ ] `list`가 최신순으로 거래를 출력하는가
- [ ] `list --limit N`이 기본값과 제한 값을 올바르게 처리하는가
- [ ] `search`가 `--from`, `--to`, `--category`, `--type`, `--q`, `--tag` 조건을 조합으로 지원하는가
- [ ] `search`가 최신순으로 결과를 출력하는가
- [ ] `summary`가 월별 수입/지출/잔액을 계산하는가
- [ ] `summary`가 TOP N 카테고리 지출을 출력하는가
- [ ] `summary`가 데이터가 없는 달에 `데이터 없음`을 출력하는가
- [ ] `update`가 존재하지 않는 ID를 올바르게 처리하는가
- [ ] `update`가 수정 필드만 반영하는가
- [ ] `delete`가 존재하지 않는 ID를 올바르게 처리하는가
- [ ] `delete`가 원자적 저장 정책을 사용하는가

### 3단계 완료 기준: 보너스 기능
- [ ] `budget set`이 예산을 저장하는가
- [ ] `summary`가 예산 사용률과 초과 경고를 표시하는가
- [ ] `category add`가 카테고리를 저장하는가
- [ ] `category list`가 등록된 카테고리를 출력하는가
- [ ] `category remove`가 사용 중인 카테고리를 보호하는가
- [ ] `import`가 CSV 파일을 파싱하고 거래를 추가하는가
- [ ] `export`가 필터 조건에 맞는 거래를 CSV로 저장하는가
- [ ] `backup`이 타임스탬프가 포함된 백업 파일을 생성하는가
- [ ] `recurring add`가 반복 규칙을 저장하는가
- [ ] `recurring run --month YYYY-MM`이 해당 월의 거래를 생성하는가

### 4단계 완료 기준: 안정성 및 오류 처리
- [ ] 입력 검증이 서비스 호출 이전에 실행되는가
- [ ] 잘못된 날짜/금액/타입/카테고리에 대해 사용자 친화 메시지를 출력하는가
- [ ] 파일 읽기/쓰기 실패 시 사용자 친화 메시지를 출력하는가
- [ ] 저장 실패 시 원본 데이터가 손상되지 않는가
- [ ] 종료 코드가 정책에 맞게 반환되는가
- [ ] `with_atomic_write` 또는 동등한 원자적 저장 전략이 적용되었는가

### 완료 기준
- [ ] 전체 테스트가 통과하는가
- [ ] README와 구현이 일치하는가
- [ ] 예외 처리와 종료 코드 정책이 일치하는가
- [ ] CLI, 서비스, Repository, Validator 테스트가 모두 존재하는가

## 9. 테스트 케이스 예시

### 9.1 테스트 폴더 구조
```text
tests/
├── __init__.py
├── conftest.py
├── test_cli.py
├── test_services.py
├── test_repositories.py
├── test_validators.py
├── fixtures/
│   ├── sample_transactions.jsonl
│   ├── sample_categories.jsonl
│   ├── sample_budgets.jsonl
│   └── sample_import.csv
```

### 9.2 공통 테스트 픽스처 및 `conftest.py` 예시
- `conftest.py`는 `tmp_path` 기반의 테스트용 작업 디렉터리를 생성하고, 샘플 파일을 복사한다.
- 테스트는 실제 파일 경로를 변경하지 않고 `cwd`를 `tmp_path`로 바꿔 실행한다.
- 공통 헬퍼는 CLI 실행, 파일 생성, JSONL 파싱을 담당한다.

```python
# tests/conftest.py
import json
import shutil
from pathlib import Path

import pytest


@pytest.fixture
def project_root() -> Path:
    return Path(__file__).resolve().parents[1]


@pytest.fixture
def fixtures_dir(project_root: Path) -> Path:
    return project_root / 'tests' / 'fixtures'


@pytest.fixture
def temp_workspace(tmp_path: Path, fixtures_dir: Path) -> Path:
    workspace = tmp_path / 'workspace'
    workspace.mkdir()

    (workspace / 'data').mkdir()
    (workspace / 'backups').mkdir()

    for fixture_name in [
        'sample_transactions.jsonl',
        'sample_categories.jsonl',
        'sample_budgets.jsonl',
        'sample_import.csv',
    ]:
        shutil.copy(fixtures_dir / fixture_name, workspace / 'data' / fixture_name)

    return workspace


@pytest.fixture
def run_cli(temp_workspace: Path):
    import subprocess
    import sys

    def _run_cli(*args: str):
        return subprocess.run(
            [sys.executable, 'main.py', *args],
            capture_output=True,
            text=True,
            cwd=temp_workspace,
        )

    return _run_cli


@pytest.fixture
def read_jsonl_file(temp_workspace: Path):
    def _read_jsonl(filename: str):
        path = temp_workspace / 'data' / filename
        with path.open('r', encoding='utf-8') as f:
            return [json.loads(line) for line in f if line.strip()]

    return _read_jsonl
```

### 9.3 CLI 테스트용 샘플 fixture 예시
#### `tests/fixtures/sample_transactions.jsonl`
```jsonl
{"id": "tx_20260501_0001", "date": "2026-05-01", "type": "income", "category": "salary", "amount": 300000, "memo": "월급", "tags": "payday"}
{"id": "tx_20260503_0001", "date": "2026-05-03", "type": "expense", "category": "food", "amount": 12000, "memo": "점심", "tags": "meal,work"}
{"id": "tx_20260510_0001", "date": "2026-05-10", "type": "expense", "category": "food", "amount": 8000, "memo": "커피", "tags": "coffee"}
```

#### `tests/fixtures/sample_categories.jsonl`
```jsonl
{"name": "salary"}
{"name": "food"}
{"name": "transport"}
```

#### `tests/fixtures/sample_budgets.jsonl`
```jsonl
{"month": "2026-05", "amount": 200000}
```

#### `tests/fixtures/sample_import.csv`
```csv
date,type,category,amount,memo,tags
2026-05-12,expense,food,15000,저녁,meal
2026-05-15,expense,transport,5000,지하철,commute
```

### 9.4 실제 pytest 파일 예시

#### `tests/test_repositories.py`
```python
import json

from app.repositories.transaction_repository import TransactionRepository
from app.models.transaction import Transaction


def test_append_writes_transaction_to_jsonl(tmp_path):
    path = tmp_path / 'transactions.jsonl'
    repo = TransactionRepository(path=path)

    transaction = Transaction(
        id='tx_1',
        date='2026-05-23',
        type='expense',
        category='food',
        amount=12000,
        memo='점심',
        tags='meal,work',
    )

    repo.append(transaction)

    lines = path.read_text(encoding='utf-8').strip().splitlines()

    assert len(lines) == 1
    saved = json.loads(lines[0])
    assert saved['id'] == 'tx_1'
    assert saved['category'] == 'food'
    assert saved['amount'] == 12000


def test_filter_returns_matching_transactions(tmp_path):
    path = tmp_path / 'transactions.jsonl'
    repo = TransactionRepository(path=path)

    repo.append(Transaction(id='tx_1', date='2026-05-01', type='expense', category='food', amount=1000, memo='a', tags=''))
    repo.append(Transaction(id='tx_2', date='2026-05-02', type='income', category='salary', amount=5000, memo='b', tags=''))
    repo.append(Transaction(id='tx_3', date='2026-05-03', type='expense', category='food', amount=2000, memo='c', tags=''))

    result = repo.filter({'category': 'food', 'type': 'expense'})

    assert len(result) == 2
    assert all(item.category == 'food' and item.type == 'expense' for item in result)
```

#### `tests/test_services.py`
```python
import pytest

from app.errors.app_error import AppError
from app.services.transaction_service import TransactionService
from app.repositories.transaction_repository import TransactionRepository
from app.repositories.category_repository import CategoryRepository


@pytest.fixture
def service(tmp_path):
    transaction_repo = TransactionRepository(path=tmp_path / 'transactions.jsonl')
    category_repo = CategoryRepository(path=tmp_path / 'categories.jsonl')
    category_repo.append({'name': 'food'})

    return TransactionService(transaction_repo=transaction_repo, category_repo=category_repo)


def test_add_transaction_persists_valid_transaction(service):
    result = service.add_transaction(
        {
            'date': '2026-05-23',
            'type': 'expense',
            'category': 'food',
            'amount': 12000,
            'memo': '점심',
            'tags': 'meal,work',
        }
    )

    assert result.id is not None
    assert result.category == 'food'
    assert result.amount == 12000


def test_add_transaction_rejects_zero_amount(service):
    with pytest.raises(AppError) as exc_info:
        service.add_transaction(
            {
                'date': '2026-05-23',
                'type': 'expense',
                'category': 'food',
                'amount': 0,
                'memo': '점심',
                'tags': 'meal,work',
            }
        )

    assert '양수' in str(exc_info.value)
```

#### `tests/test_validators.py`
```python
import pytest

from app.validators.transaction_validator import validate_date, validate_amount
from app.errors.app_error import AppError


@pytest.mark.parametrize('value', ['2026/05/23', '2026-13-01', 'not-a-date'])
def test_validate_date_rejects_invalid_format(value):
    with pytest.raises(AppError):
        validate_date(value)


@pytest.mark.parametrize('value', [0, -1])
def test_validate_amount_rejects_non_positive(value):
    with pytest.raises(AppError):
        validate_amount(value)
```

#### `tests/test_cli.py`
```python
import json
from pathlib import Path


def test_list_limit_two_outputs_two_records(run_cli, temp_workspace):
    result = run_cli('list', '--limit', '2')

    assert result.returncode == 0
    assert result.stdout.count('2026-05-') == 2


def test_summary_outputs_income_expense_and_balance(run_cli, temp_workspace):
    result = run_cli('summary', '--month', '2026-05', '--top', '2')

    assert result.returncode == 0
    assert '총수입' in result.stdout
    assert '총지출' in result.stdout
    assert '잔액' in result.stdout


def test_delete_unknown_id_returns_user_friendly_error(run_cli, temp_workspace):
    result = run_cli('delete', '--id', 'missing-id')

    assert result.returncode != 0
    assert '오류:' in result.stdout
    assert '해결:' in result.stdout


def test_import_csv_adds_transactions(run_cli, read_jsonl_file, temp_workspace):
    result = run_cli('import', '--from', 'data/sample_import.csv')

    assert result.returncode == 0
    assert 'import 완료' in result.stdout

    records = read_jsonl_file('transactions.jsonl')
    assert len(records) >= 2
    assert records[-1]['category'] == 'transport'
```

### 9.5 실제 테스트 실행 순서
1. `pytest tests/test_repositories.py`
2. `pytest tests/test_services.py`
3. `pytest tests/test_validators.py`
4. `pytest tests/test_cli.py`
5. `pytest`

### 9.6 테스트 작성 원칙
- 각 테스트는 파일 시스템을 고립된 `tmp_path` 또는 `temp_workspace` 기반으로 실행한다.
- CLI 테스트는 `subprocess`가 아니라 `run_cli` 헬퍼를 통해 실행한다.
- 서비스 테스트는 Repository를 직접 주입해 의존성을 통제한다.
- 예외 메시지는 사용자 친화 문구와 해결 힌트가 포함되는지까지 검증한다.

## 10. 권장 문서/코드 작성 순서

1. README를 기준으로 구현 범위를 고정한다.
2. `main.py`와 CLI 경로를 먼저 구성한다.
3. 모델과 저장소를 작성한다.
4. 서비스 레이어를 구현한다.
5. 검증과 에러 처리를 연결한다.
6. 테스트를 작성하고 반복 검증한다.
7. README의 체크리스트를 완료 상태로 맞춘다.

## 10. 결론

실제 프로젝트를 진행할 때는 **구조 먼저**, **데이터 저장소 먼저**, **핵심 CRUD 먼저**, **보너스 기능 나중**, **테스트 우선** 순서로 진행하는 것이 가장 안정적이다.

이 순서를 따르면 README에 있는 기능 명세와 구현이 쉽게 맞물리고, 테스트도 단계적으로 확장할 수 있다.
