# scoda-engine-core (독립 Core 패키지)

`scoda-engine-core`는 SCODA 패키지 로딩/검증의 핵심 로직을 담은 **stdlib-only 독립 PyPI 패키지**. 2026-02-21에 `scoda-engine` 레포 내 모노레포 구조(`core/`)로 분리되었다.

## 배경

2026-02-19에 scoda-engine이 trilobase에서 독립한 직후, 향후 로드맵(P01) 수립 과정에서 SCODA 메커니즘을 하나의 얇은 런타임 의존성으로 추출하자는 아이디어가 나왔다. 이를 통해:

- 외부 프로젝트가 `pip install scoda-engine-core`만으로 `.scoda` 패키지를 읽을 수 있음
- scoda-engine (Desktop) 본체는 UI/서버 의존성에 집중
- `validate_manifest.py` trilobase 중복 제거 (S-3)

## 레포지토리 레이아웃 (모노레포)

```
scoda-engine/
├── core/                              # 독립 PyPI 패키지 "scoda-engine-core"
│   ├── pyproject.toml                 # dependencies = []  (stdlib 전용)
│   └── scoda_engine_core/
│       ├── __init__.py                # Public API re-export
│       ├── scoda_package.py           # ScodaPackage, PackageRegistry
│       └── validate_manifest.py       # manifest 검증 (S-3)
├── pyproject.toml                     # scoda-engine — scoda-engine-core>=0.1.0 의존
└── scoda_engine/
    └── scoda_package.py               # 7줄 sys.modules shim (하위호환)
```

## S-2: Core 패키지 분리 (2026-02-21)

P06 계획서의 10단계 실행:

### 1. 파일 이동
- `scoda_engine/scoda_package.py` (789줄) → `core/scoda_engine_core/scoda_package.py`
- `base_dir` 경로 계산을 dirname 3단계로 수정 (모노레포 레이아웃 대응)

### 2. sys.modules Shim (하위호환)
- 기존 코드(`from scoda_engine.scoda_package import ...`)를 깨뜨리지 않기 위해 7줄짜리 shim 파일 유지:
  ```python
  import sys, scoda_engine_core
  sys.modules[__name__] = scoda_engine_core
  ```

### 3. Import 마이그레이션 (7개 파일)
| 파일 | 변경 |
|---|---|
| `scoda_engine/app.py` | `from .scoda_package import` → `from scoda_engine_core import` |
| `scoda_engine/mcp_server.py` | 동일 |
| `scoda_engine/gui.py` | `from . import scoda_package` → `import scoda_engine_core as scoda_package` (+ logger handler 추가) |
| `scoda_engine/serve.py` | 동일 |
| `scripts/release.py` | 동일 |
| `tests/conftest.py` | 동일 |
| `tests/test_runtime.py` | 내부 변수 접근은 `scoda_engine_core.scoda_package` 직접 import |

### 4. 설정 변경
- scoda-engine `pyproject.toml`에 `scoda-engine-core>=0.1.0` 의존성 추가, `exclude = ["core*"]`
- `ScodaDesktop.spec` (PyInstaller): 양쪽 exe에 `scoda_engine_core` hiddenimports + datas 추가

### 5. 결과
- 196 tests passing
- PyInstaller 단일 EXE 정상 동작 (exe 빌드 후 여러 문제 순차 수정)

## S-3: validate_manifest 중복 제거 (2026-02-22)

- trilobase와 scoda-engine 양쪽에 존재하던 `validate_manifest.py`를 core로 이동
- trilobase 측은 `from scoda_engine_core import validate_manifest`로 교체 (P10)
- 225 tests passing

## 버전 관리 전략 (P07, 2026-02-21)

- **독립 SemVer**: `scoda-engine-core`와 `scoda-engine` 각각 별도 버전
- **Git 태그 prefix 분리**:
  - Core: `core-v0.1.0`, `core-v0.1.1`, ...
  - Desktop: `desktop-v0.1.0`, `desktop-v0.1.1`, ...
- **Release workflow**: 태그 prefix 기반으로 빌드 분기
- 초기 버전: core `0.1.0` (02-21) → `0.1.1` (02-22, validate_manifest 통합)

## Generic Fixture (S-1, 2026-02-20)

Core 분리 직전에 테스트 인프라를 도메인 독립적으로 전환:

- **`generic_db` / `generic_client`**: categories, items, tags, relations 범용 스키마
- **`generic_dep_db`**: regions, locations, time_periods (의존성 해석 테스트용)
- **`conftest.py` 1,975 → 792줄** (trilobase 특화 제거)
- **`release.py` 범용화**: hardcoded trilobase 참조 제거, `--name` CLI로 패키지명 주입

이렇게 해서 core 패키지가 어떤 도메인 패키지에도 바인딩되지 않도록 보장했다.

## Public API

```python
from scoda_engine_core import (
    ScodaPackage,         # .scoda 파일 로드, manifest 파싱, 의존성 해석
    PackageRegistry,      # 패키지 스캔/등록/조회
    validate_manifest,    # manifest 스키마 검증
    # 예외 (S-5에서 추가)
    ScodaDependencyError,
    ScodaChecksumError,
    ScodaManifestError,
)
```

## 관련 페이지

- [scoda-engine](scoda-engine.md) — Core를 사용하는 런타임
- [SCODA 아키텍처](scoda.md) — 패키지 규격
- [릴리스 워크플로우](scoda-engine-release.md) — 독립 SemVer 릴리스
- [Meta-Package](meta-package.md) — Core의 `kind` 필드 지원

---
*Sources: P01, P06, P07, 004, P09, P10, 005, S-1 (001/002, P02/P03).*
