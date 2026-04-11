# scoda-engine (SCODA 뷰어/서버 런타임)

모든 SCODA 패키지를 렌더링하는 범용 뷰어/서버 런타임. 2026-02-19에 trilobase에서 독립 저장소 `scoda-engine`으로 분리된 후, 25일(2026-02-19 ~ 2026-03-18) 동안 집중 개발되어 v0.1.x → v0.3.4+까지 진화했다.

## 개요

| 항목 | 값 |
|---|---|
| 패키지명 | `scoda-engine` (import: `scoda_engine`), `scoda-engine-core` (import: `scoda_engine_core`) |
| 설치 | `pip install -e ".[dev]"` (모노레포 `core/` 서브패키지 포함) |
| 진입점 | `scoda_engine.gui` (Desktop GUI), `scoda_engine.serve` (CLI), `scoda_engine.serve_web` (Docker) |
| 테스트 | 191 → 303+ (25일간 추가) |
| 최신 버전 | Desktop `0.3.4+`, Core `0.1.x`, Server (Docker) `0.1.x` |

## 레포지토리 구조 (모노레포)

```
scoda-engine/
├── core/                          # scoda-engine-core (독립 PyPI 패키지)
│   ├── pyproject.toml
│   └── scoda_engine_core/
│       ├── scoda_package.py       # ScodaPackage, PackageRegistry
│       └── validate_manifest.py   # manifest 검증 (S-3에서 core로 이동)
├── scoda_engine/
│   ├── app.py                     # FastAPI (Flask에서 마이그레이션 완료)
│   ├── gui.py / launcher_gui.py   # tkinter Desktop GUI
│   ├── serve.py                   # CLI 진입점
│   ├── serve_web.py               # gunicorn 팩토리 (프로덕션 Docker)
│   ├── mcp_server.py              # MCP stdio 런처
│   ├── scoda_package.py           # 7줄 sys.modules shim (core 재export)
│   ├── hub_client.py              # Hub 클라이언트 (fetch/compare/download)
│   ├── entity_schema.py           # CRUD 프레임워크 스키마 정의
│   ├── crud_engine.py             # 제네릭 CRUD 실행 엔진
│   ├── static/
│   │   ├── js/app.js              # SPA 메인 라우터
│   │   ├── js/tree_chart.js       # TreeChartInstance (D3 + Canvas)
│   │   ├── vendor/                # D3/Bootstrap/BI 오프라인 번들
│   │   └── css/style.css
│   └── templates/
│       ├── index.html
│       └── landing.html           # Multi-package / meta-package 랜딩
├── hub/
│   ├── sources.json               # 수집 대상 repo (수동 관리)
│   └── (scoda-hub-index.json은 auto-generated, gh-pages 배포)
├── deploy/
│   ├── Dockerfile                 # 단일 컨테이너 (nginx 제거 완료)
│   ├── docker-compose.yml
│   └── fetch_packages.py          # 빌드 타임 Hub 패키지 다운로드
├── scripts/
│   ├── generate_hub_index.py      # GitHub Releases → Hub index 빌드
│   ├── bump_version.py            # pyproject + __init__ + compose 동시 갱신
│   └── release.py                 # PyInstaller 빌드 (범용화 완료)
├── .github/workflows/
│   ├── test.yml                   # 2 OS × 2 Python 매트릭스 CI
│   ├── release.yml                # PyInstaller + GitHub Release + Docker Hub
│   ├── hub-index.yml              # Hub index 자동 생성 + gh-pages 배포
│   └── pages.yml                  # MkDocs + Hub index 통합 배포
├── docs/                          # MkDocs + Material (EN/KO)
└── tests/                         # generic fixture 기반 (S-1 완료)
```

## 주요 개발 마일스톤 (S-1 ~ S-7)

| # | 주제 | 주요 산출물 |
|---|---|---|
| **S-1** (02-20) | 테스트 fixture 범용화 | `generic_db`/`generic_client`/`generic_dep_db` fixture, conftest 1,975→792줄, release.py 범용화 |
| **S-2** (02-21) | Core 패키지 분리 | `scoda_engine_core` 모노레포, `sys.modules` 하위호환 shim, 독립 SemVer + `core-v*`/`desktop-v*` 태그 |
| **S-3** (02-22) | `validate_manifest` 중복 제거 | trilobase 측 중복 제거, core로 통합 |
| **S-5** (02-22) | SCODA Spec Alignment | checksum-on-load, SemVer range 파싱, `required` dependency, `ScodaDependencyError`, CHANGELOG.md |
| **S-6** (02-23) | `label_map` 동적 컬럼 | 행 데이터 기반 헤더 결정 (`renderLinkedTable`) |
| **S-7** (02-23) | CI/CD | `test.yml` 매트릭스, `release.yml` PyInstaller, 이후 Docker Hub 자동 빌드 추가 |

## 핵심 하위 시스템

### Desktop GUI 런처 (`gui.py` / `launcher_gui.py`)

- tkinter 기반: Start/Stop, DB 상태, 자동 브라우저 열기, 실시간 로그 뷰어
- 구조화 로깅 (JSON 기반 핸들러)
- **임의 경로 로딩** (02-24): `PackageRegistry.register_path()`, `--scoda-path` CLI, `SCODA_PACKAGE_PATH` 환경변수
- **서버 포트 설정/자동 탐색** (02-26): Hyper-V 포트 충돌 대응, `ScodaDesktop.cfg`에 저장
- **Hub 섹션**: 백그라운드 체크, 프로그레스 바, SHA-256 검증, "Check Hub" 리프레시 버튼
- **멀티 사이즈 ICO 아이콘** (02-25)

### FastAPI 서버 (`app.py`)

- **Flask → FastAPI 마이그레이션 완료**: 비동기 지원, Pydantic 응답 모델, OpenAPI 자동 문서
- 동적 MCP 도구 생성: `ui_queries`에서 런타임 생성 (→ [MCP 서버](mcp-server.md))
- `/api/query` 복합 엔드포인트
- `/healthz` 엔드포인트 (프로덕션 Docker)
- **Multi-Package Serving** (v0.3.0, P29): `APIRouter(prefix="/api/{package}")` 라우팅 → [multi-package-serving.md](multi-package-serving.md)
- **Meta-Package API** (v0.3.4+, P31): `/api/{pkg}/meta/tree`, `/meta/bindings`, `/meta/composite-tree` → [meta-package.md](meta-package.md)
- **Hub Refresh API**: `POST /api/hub/sync` (온디맨드) + 주기적 daemon sync

### 범용 뷰어 SPA

- Bootstrap 5 기반, CORS 지원, 브라우저 히스토리/뒤로가기
- 매니페스트 기반 자동 뷰 렌더링 (→ [SCODA 아키텍처](scoda.md))
- 글로벌 검색박스, Global Controls 프레임워크 (manifest-driven 드롭다운, 쿼리 자동 병합)
- Preferences API (overlay DB 저장, localStorage 완전 제거)
- Home 브레드크럼 (🏠 SCODA / PackageName) + 랜딩 페이지 D3 force 배경

### 뷰 타입 (manifest `display_type`)

| 뷰 | 설명 |
|---|---|
| `tree` / `radial` / `rectangular` | Tree Chart — 계층 트리 (D3 cluster + Canvas+SVG LOD) |
| `table` | 데이터 테이블 |
| `detail` | 상세 페이지 (`linked_table` 섹션) |
| `chart` | ICS 지질연대 차트 |
| `timeline` | 시간대별 분포 |
| `bar_chart` | Stacked bar chart (2026-03-17 추가) |
| `compound` | Compound View (로컬 컨트롤 + 서브탭) |
| `tree_chart_morph` | Morph animation 서브뷰 |
| `tree_chart_timeline` | Timeline 서브뷰 (연쇄 morph + 녹화) |

→ 상세: [시각화](visualization.md)

### CRUD 프레임워크 (v0.1.3, P21)

- manifest-driven: `entity_schema.py` + `crud_engine.py`
- REST API 10개 엔드포인트 (`/api/entities/*`, `/api/search/*`)
- Admin/Viewer 모드 분리, FK autocomplete, `readonly_on_edit` 필드
- 27개 CRUD 테스트

→ 상세: [CRUD 프레임워크](crud-framework.md)

### SCODA Hub 클라이언트

- `hub_client.py` (순수 stdlib): fetch/compare/download/resolve
- SSL fallback (기관 프록시, Windows 인증서 저장소, `HubSSLError`)
- Dependency UI + 다운로드 확인 다이얼로그
- Hub 파일명: `scoda-hub-index.json` (MkDocs와의 충돌 해소)

→ 상세: [SCODA Hub](scoda-hub.md)

### 프로덕션 Docker (`serve_web.py` + `deploy/`)

- gunicorn 팩토리, 단일 컨테이너 (nginx 완전 제거), 포트 8081
- `fetch_packages.py`로 빌드 타임 Hub 패키지 자동 다운로드
- `SCODA_ENGINE_NAME` — Desktop/Server 구분 표시
- Docker Hub `honestjung/scoda-server:X.Y.Z` 자동 게시 (release workflow)

→ 상세: [Docker 배포](docker-deployment.md)

### 릴리스 인프라

- `release.py` 범용화 (hardcoded trilobase 참조 제거)
- PyInstaller 단일 EXE (GUI + Desktop 서버 + MCP 통합)
- `bump_version.py`: `pyproject.toml` + `__init__.py` + `deploy/docker-compose.yml` 동시 갱신 (버전 불일치 방지)
- MkDocs + Material 테마 다국어(EN/KO) 문서 사이트 (P19)

→ 상세: [릴리스 워크플로우](scoda-engine-release.md)

## 버전 이력 (v0.1.1 ~ v0.3.4+)

| 날짜 | Desktop 버전 | 주요 변경 |
|---|---|---|
| 02-22 | core 0.1.1 | validate_manifest 통합 |
| 02-24 | 0.1.1 | 임의 경로 `.scoda` 로딩 |
| 02-24 | 0.1.2 | Hub 자동 체크/다운로드, Open File/D&D 제거 |
| 03-01 | 0.1.3 | Radial hierarchy display, CRUD 프레임워크, Preferences API |
| 03-02 | server 0.1.0 | 프로덕션 Docker 배포 (Docker Hub 게시) |
| 03-03 | 0.1.5 | Tree Chart bottom-up 레이아웃 (leaf 겹침 원천 방지) |
| 03-06 | 0.1.7 / 0.1.8 | Rectangular leaf 간격 최적화 + Docker Hub 자동화 |
| 03-07 | 0.2.0 | `TreeChartInstance` 리팩토링 + Side-by-Side 뷰 + Diff Tree |
| 03-09 | 0.2.x | Compound View + Morph animation |
| 03-11 | 0.2.3 | Watch / Removed 패널, Hub latest-only, UI 개선 |
| 03-12 | 0.2.x | Hub Refresh 버튼, 모바일 햄버거 메뉴, Morph WebM 녹화 |
| 03-14 | 0.2.5 / 0.3.0 | Timeline sub-view + Vendor JS 내장 → **Multi-Package Serving** |
| 03-15 | 0.3.1 / 0.3.2 | 모바일 UI (Landing 스크롤, Tree 드로어, Removed 접기, Compound 서브탭) |
| 03-17 | 0.3.3 / 0.3.4 | Timeline 모바일 UX, Timeline WebM 녹화, **Bar Chart** subview, Statistics 탭 재편 |
| 03-18 | 0.3.4+ | **Meta-Package 지원** (paleobase), D3 radial tree, top-level 자동 리다이렉트 |

## 설계 문서 (미구현/부분 구현)

| 문서 | 주제 | 상태 |
|---|---|---|
| P01 | Future Work Roadmap (S-1 ~ S-4) | S-1~S-3 완료, S-4(SCODA 백오피스) 대기 |
| P05 | SCODA Distribution and Architecture Strategy | 부분 구현 |
| P11 | Tree Snapshot Design v1 Review (2-layer 아키텍처) | 설계 검토 완료, 구현 대기 |

## 관련 페이지

- [SCODA 아키텍처](scoda.md) — 패키지 규격, manifest
- [scoda-engine-core](scoda-engine-core.md) — 독립 PyPI 패키지 분리
- [시각화](visualization.md) — Tree Chart 엔진 상세
- [SCODA Hub](scoda-hub.md) — 정적 레지스트리 + 클라이언트
- [CRUD 프레임워크](crud-framework.md) — manifest-driven CRUD
- [Multi-Package Serving](multi-package-serving.md) — URL prefix 라우팅
- [Meta-Package](meta-package.md) — paleobase 합성 트리
- [Docker 배포](docker-deployment.md) — 프로덕션 서버
- [릴리스 워크플로우](scoda-engine-release.md) — CI/CD, PyInstaller, MkDocs
- [MCP 서버](mcp-server.md) — LLM 통합
- [Trilobase 개요](trilobase-overview.md) — scoda-engine의 첫 소비자

---
*Sources: raw/scoda-engine-devlog/ 전체 97 파일 (2026-02-19 ~ 2026-03-18). 추가 컨텍스트는 raw/trilobase-devlog/ 아카이브 참조.*
