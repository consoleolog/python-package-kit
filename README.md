# Python Package Kit

GitHub Issue 기반 협업 워크플로우를 빠르게 적용할 수 있는 Python 라이브러리 보일러플레이트입니다.

---

## 목차

1. [구성 요소](#구성-요소)
2. [시작하기](#시작하기)
3. [브랜치 구조](#브랜치-구조)
4. [개발 흐름](#개발-흐름)
5. [자동화](#자동화)
6. [실행 명령](#실행-명령)
7. [배포](#배포)
8. [가이드](#가이드)

---

## 구성 요소

```
.
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── config.yml                  # 빈 이슈 생성 비활성화
│   │   ├── FEATURE.md                  # 기능 추가 이슈 템플릿
│   │   ├── BUG.md                      # 버그 수정 이슈 템플릿
│   │   ├── REFACTOR.md                 # 리팩토링 이슈 템플릿
│   │   ├── SETUP.md                    # 프로젝트 초기 세팅 이슈 템플릿
│   │   └── HOTFIX.md                   # 긴급 버그 수정 이슈 템플릿
│   ├── workflows/
│   │   ├── create-branch-on-issue.yml  # Issue Assign 시 브랜치 자동 생성
│   │   └── label-sync.yml              # 라벨 동기화
│   └── issue-branch.yml                # 브랜치 자동 생성 규칙
├── docs/
│   ├── configs/
│   │   ├── BANDIT.md                   # bandit 보안 검사 설정 가이드
│   │   ├── PYTEST.md                   # pytest 설정 가이드
│   │   └── RUFF.md                     # ruff 린트/포맷 설정 가이드
│   └── guides/
│       ├── COMMIT.md                   # 커밋 메시지 작성 가이드
│       ├── GIT_WORKFLOW.md             # Git 워크플로우 가이드
│       └── CODE_REVIEW.md              # 코드 리뷰 가이드
├── src/
│   └── <package_name>/
│       └── __init__.py                 # 라이브러리 진입점
├── tests/
│   ├── __init__.py
│   └── conftest.py                     # 공용 pytest fixture
├── .env.example                        # 환경 변수 예시
├── .bandit.yaml                        # bandit 보안 검사 설정
├── .ruff.toml                          # ruff 린트/포맷 설정
├── hatch.toml                          # hatch 빌드/버전/스크립트 설정
├── lefthook.yml                        # pre-commit 훅 설정
├── pyproject.toml                      # 프로젝트 메타데이터 및 의존성
└── pytest.ini                          # pytest 설정
```

---

## 시작하기

### 1. 패키지 이름 설정

`pyproject.toml`의 `name`, `hatch.toml`의 `packages` 경로, `src/<package_name>/` 폴더명을 실제 패키지 이름으로 변경합니다.

### 2. 레포지토리 생성

이 레포지토리를 템플릿으로 사용하거나 파일을 복사합니다.

### 3. `dev` 브랜치 생성

```bash
git checkout -b dev
git push origin dev
```

### 4. 기본 브랜치 설정

GitHub 레포지토리 설정에서 기본 브랜치를 `dev`로 변경합니다.

> `Settings` → `General` → `Default branch` → `dev`

### 5. 의존성 설치

```bash
uv sync
```

### 6. pre-commit 훅 등록

```bash
lefthook install
```

---

## 브랜치 구조

| 브랜치 | 역할 |
|--------|------|
| `main` | 운영(배포) 환경, 항상 안정적인 상태 유지 |
| `dev` | 개발 통합 브랜치, 모든 기능 개발의 기준점 |
| `feature/dev-<n>` | 기능 개발, `dev`에서 분기 |
| `hotfix/main-<n>` | 운영 중 발생한 긴급 버그 수정, `main`에서 분기 |

```
main ◄──────────────── hotfix/main-<n>
 ▲                            │
 │                            ▼ (dev에도 반영)
dev ◄─── feature/dev-<n>    dev
```

---

## 개발 흐름

### 기능 개발 (feature)

1. **Issue 생성** — 작업을 Issue로 등록합니다.
2. **Issue Assign** — 본인에게 Assign하면 브랜치가 자동 생성됩니다.
3. **작업 및 커밋** — [커밋 가이드](docs/guides/COMMIT.md) 규칙에 따라 커밋합니다.
4. **PR 생성** — `dev`를 base로 PR을 생성합니다.
5. **코드 리뷰** — [코드 리뷰 가이드](docs/guides/CODE_REVIEW.md)에 따라 리뷰합니다.
6. **Rebase Merge** — 리뷰 완료 후 `dev`에 Rebase Merge합니다.

### 릴리즈 (dev → main)

1. **PR 생성** — `dev` → `main` PR을 생성합니다. (제목: `release: v1.0.0`)
2. **리뷰 및 승인** — 배포 범위와 변경사항을 최종 확인합니다.
3. **Rebase Merge** — GitHub UI에서 **"Rebase and merge"** 로 병합합니다.
4. **Git Tag** — 머지 후 배포 버전을 태그로 기록합니다.

### 긴급 버그 수정 (hotfix)

1. `main`에서 `hotfix/main-<n>` 브랜치 생성
2. 수정 후 `main`으로 PR → Rebase Merge
3. `main` 변경사항을 `dev`에도 반영

---

## 자동화

### Issue Assign → 브랜치 자동 생성

Issue를 본인에게 Assign하면 `issue-branch.yml` 규칙에 따라 브랜치가 자동 생성됩니다.

| label                                     | 생성 브랜치               |
|-------------------------------------------|----------------------|
| `enhancement`, `setup`, `bug`, `refactor` | `feature/dev-<이슈번호>` |
| `hotfix`                                  | `hotfix/main-<이슈번호>` |

### pre-commit 훅 (lefthook)

커밋 시 아래 검사가 순서대로 실행됩니다.

| 잡             | 대상                   | 실행 명령                        |
|---------------|----------------------|------------------------------|
| `ruff-lint`   | `*.py`               | `ruff check`                 |
| `ruff-format` | `*.py`               | `ruff format --check`        |
| `bandit`      | `*.py`               | `bandit -q`                  |
| `test`        | `tests/**/test_*.py` | `pytest --no-cov -n auto -q` |

훅을 수동으로 실행하려면:

```bash
lefthook run pre-commit
```

---

## 실행 명령

### 린트 & 포맷

```bash
uv run ruff check .          # 린트 검사
uv run ruff check --fix .    # 린트 자동 수정
uv run ruff format --check . # 포맷 검사
uv run ruff format .         # 포맷 적용
```

### 보안 검사

```bash
uv run bandit -r src/
```

### 테스트

```bash
uv run pytest                      # 전체 테스트 (커버리지 포함)
uv run pytest --no-cov             # 커버리지 제외
uv run pytest --no-cov -n auto     # 병렬 실행
```

### hatch 스크립트

```bash
hatch run test          # pytest 전체
hatch run test-fast     # 커버리지 제외 병렬 실행
hatch run lint          # ruff check
hatch run lint-fix      # ruff 자동 수정
hatch run format        # ruff format
hatch run format-check  # 포맷 검사
hatch run security      # bandit 보안 검사
```

---

## 배포

### PyPI 배포

```bash
# 1. main 머지 후 태그 생성 (hatch-vcs가 태그에서 버전 자동 추출)
git tag v1.0.0
git push origin v1.0.0

# 2. 빌드
uv build

# 3. 배포
uv publish --token pypi-xxxx...
```

### GitHub에서 직접 설치 (PyPI 배포 없이)

```bash
# main 브랜치
pip install git+https://github.com/<org>/<package-name>.git

# 특정 태그
pip install git+https://github.com/<org>/<package-name>.git@v1.0.0

# uv 사용 시
uv add git+https://github.com/<org>/<package-name>.git@v1.0.0
```

### 설치 후 사용

```python
from <package_name> import ...
```

---

## 가이드

### 개발 가이드

- [커밋 메시지 작성 가이드](docs/guides/COMMIT.md)
- [Git 워크플로우 가이드](docs/guides/GIT_WORKFLOW.md)
- [코드 리뷰 가이드](docs/guides/CODE_REVIEW.md)

### 설정 가이드

- [ruff 린트/포맷 설정](docs/configs/RUFF.md)
- [bandit 보안 검사 설정](docs/configs/BANDIT.md)
- [pytest 설정](docs/configs/PYTEST.md)