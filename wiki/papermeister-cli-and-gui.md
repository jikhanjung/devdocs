# PaperMeister CLI & Desktop GUI

PaperMeister의 사용자 경로. **두 가지 진입점**을 제공한다:
- **CLI** (PyQt 불필요) — 자동화, 헤드리스 처리, 초기 구축에 우선
- **PyQt6 Desktop GUI** — 일상 사용, 탐색, 제어

2026-04-09 ~ 04-10의 **P06 Desktop MVP Feature Definition**과 **P07 Desktop Software Implementation Plan**에서 현재 데스크톱 앱의 목표 scope와 구현 순서가 정립되었다.

## 초기 PyQt6 GUI (P01, 001)

### MVP 레이아웃

```
+------------------------------------------------------------+
| PaperMeister                                               |
+------------------------------------------------------------+
| File(Import Folder, Exit)                                  |
+------------------------------------------------------------+
| [Search...                              ] [Search][Show All]|
+------------------------------------------------------------+
|  Results (TreeWidget)     |  Detail (TextEdit, readonly)   |
|                           |                                 |
|  - Paper Title 1  2020  3 |  Title / Authors / Year / DOI  |
|  - Paper Title 2  2018  1 |  Matching Passages             |
|  - Paper Title 3  2023  5 |  Full Text by Page             |
+------------------------------------------------------------+
| Status: Papers: N | Passages: M                            |
+------------------------------------------------------------+
```

### 초기 모듈

```
papermeister/
├── __init__.py
├── models.py          # Peewee: Paper, Author, PaperFile, Passage
├── database.py        # DB 초기화 + FTS5 가상 테이블
├── ingestion.py       # 디렉토리 스캔, 해시 dedup, PDF 등록
├── text_extract.py    # PyMuPDF 텍스트/메타데이터, passage 분할
├── search.py          # FTS5 BM25 + snippet()
└── ui/
    ├── __init__.py
    └── main_window.py # PyQt6 메인 윈도우
main.py                # 엔트리포인트
```

### 초기 검색 전략 (search.py)

SQLite FTS5 가상 테이블 `passage_fts`:

| 컬럼 | 가중치 | 용도 |
|---|---|---|
| `title` | 10 | 제목 매칭 |
| `authors` | 5 | 저자 매칭 |
| `text` | 1 | 본문 매칭 |
| `paper_id` | UNINDEXED | 조인용 |
| `page` | UNINDEXED | 페이지 |
| `passage_id` | UNINDEXED | 상세 이동 |

BM25 랭킹, `snippet()`으로 매칭 구절 하이라이트.

### 데이터 흐름 (초기)

```
[폴더 선택] → scan_directory() → ingest_pdf() (해시 dedup)
    → PaperFile(status='pending') 생성
    → process_paper_file()
        → extract_metadata_from_pdf()  → Paper 업데이트
        → extract_text_from_pdf()      → 페이지별 텍스트
        → split_into_passages()        → Passage 레코드
        → FTS5 인덱스 삽입
        → PaperFile(status='processed')
```

## CLI 구현 (006, 2026-04-01)

PyQt 의존성 없이 동작하는 CLI. 주요 명령:

| 명령 | 용도 |
|---|---|
| `import folder <path>` | 디렉토리 재귀 스캔 + PDF 등록 |
| `import zotero --collection <id>` | Zotero collection sync (→ [Zotero Integration](papermeister-zotero-integration.md)) |
| `process [--limit N]` | pending 상태 PaperFile 처리 (OCR → passage → FTS) |
| `search <query>` | FTS5 검색 + snippet 출력 |
| `list [--source ...]` | Paper/PaperFile 목록 |
| `zotero ...` | Zotero 관련 sub-commands |

### CLI의 의미

- **CI/자동화 친화** — headless 환경에서 대량 처리
- **PyQt 의존성 없이** — 서버 배포 가능
- **GUI와 동일한 core 재사용** — `papermeister/` 모듈을 그대로 호출

## CLI 개선 이력

### Collection Pagination (007)

대형 collection 처리 시 점진적 commit + progress:
- 100편 단위 iteration
- 각 페이지 받을 때마다 DB commit
- 중간 취소/재개 지원 (Zotero key 기반 skip)

→ 상세: [Zotero Integration의 Pagination 섹션](papermeister-zotero-integration.md#collection-pagination-007-2026-04-01)

### 버그픽스 + Multi-Select (009)

- 여러 collection 동시 sync (`--collection` 복수 지정)
- Collection ID 목록 파싱 버그 수정
- 부분 실패 시에도 나머지 collection은 계속 진행

## Desktop MVP Feature Definition (P06, 2026-04-09)

**목적**: "데스크탑 앱의 실제 기능을 어디까지 할 것인가"를 정책 수준에서 고정.

### 원칙

- **Searchable corpus는 OCR만으로 충분하지 않다** — bibliographic 레이어가 usability의 핵심
- **Source provenance를 유지한 UI**가 현재 데이터 모델에 더 정직하다 (canonical merge 완성 전까지 source-first 표시)
- **Vision pass, standalone promote, Zotero write-back**은 중요하지만 **정책 제어가 필요한 자동화**다 — 자동 실행 금지, 사용자 승인 흐름

### 기능 Scope

P06이 정의한 기능 카테고리:

| 카테고리 | 포함 |
|---|---|
| **Source 관리** | Zotero + 로컬 디렉토리, 등록/삭제, sync, 설정 |
| **Paper list** | 필터링, 정렬, source 표시, 처리 상태 표시 |
| **Paper detail** | 메타데이터, OCR 상태, passage, source provenance |
| **Search** | FTS5 + metadata 혼합 검색, snippet, filter |
| **Process window** | OCR queue, 진행 상태, 실패 검토 |
| **Preferences** | OCR backend, LLM 설정, Zotero API key |
| **Review queue** | standalone promote, vision pass, biblio confidence low |

## Desktop Software Implementation Plan (P07, 2026-04-10)

**가장 최신 계획 문서.** P06의 기능을 **6개 레이어**로 분해하여 구현 순서를 정한다.

### 구현 전략

1. **Foundation first** — L1 네 가지를 먼저 안정화
2. **Bibliographic usability early** — 서지정보 추출을 뒤로 미루지 않음. 검색 가능 corpus의 실질 가치가 여기서 올라가기 때문.
3. **UI는 source-first** — canonical merge가 성숙하기 전까지는 source별 구조와 operational view를 병행 제공
4. **자동화는 단계적으로** — 추출 저장이 먼저, promote/write-back/vision은 통제 가능한 흐름으로 뒤에 붙임

### 6 Layer 상세

#### L1. Corpus Foundation
- source 등록
- source 스캔 / 동기화
- PDF hash 계산
- OCR 실행
- **OCR JSON 캐시** (→ [OCR Pipeline](papermeister-ocr-pipeline.md))
- 처리 상태 추적 (`pending` / `processing` / `processed` / `failed`)

#### L2. Bibliographic Layer
- OCR JSON 기반 서지정보 추출 (→ [Biblio Extraction](papermeister-biblio-extraction.md))
- `PaperBiblio` 저장
- confidence / doc_type / needs_visual_review 처리
- Paper 반영 규칙

#### L3. UI Layer
- **source-first navigation** (source 트리가 먼저, canonical view는 나중)
- operational views (processing queue, failure list)
- paper list (filter, sort, search)
- detail panel (메타, passage, OCR 상태, provenance)
- preferences
- process window (OCR 진행 실시간)

#### L4. Search Layer
- FTS5 BM25 (기존 MVP 유지)
- **metadata + text 혼합 검색** — 제목/저자 필드 검색도 합침
- filters (source별, 연도별, 처리상태별)
- sorting (relevance, date, title)
- snippet highlight

#### L5. Controlled Automation
- **review queue** — 사용자가 검토 후 승인하는 작업 목록
  - standalone promote 후보
  - vision pass 후보
  - low confidence biblio 결과
- **vision pass** — 비싼 작업, review queue 통해서만 실행
- **standalone promote** — Zotero write-back 후보
- **Zotero write-back** — 승인 기반 실행 (→ [Zotero Integration](papermeister-zotero-integration.md#zotero-write-back-정책))

#### L6. Hardening
- retry / resume / concurrency (→ [OCR batch 018](papermeister-ocr-pipeline.md#batch-retry--resume--concurrency-018-2026-04-09))
- **failure taxonomy** — 일시적 / 영구적 / 알 수 없음
- **backend switching** — Chandra2 / Ollama / future 엔진 전환
- 암호화 PDF 대응

### 단계별 완료 기준

- L1 완료 = 여러 source의 PDF를 수집해서 안정적으로 OCR 캐싱할 수 있다
- L2 완료 = 대부분의 paper가 bibliographic 정보를 가진다 (confidence 기준 충족)
- L3 완료 = 사용자가 source-first 네비게이션으로 원하는 paper를 찾고 세부 정보를 볼 수 있다
- L4 완료 = metadata + text 혼합 검색으로 실질적인 searchable corpus 가 된다
- L5 완료 = 자동화 작업이 user-controlled 방식으로 동작한다
- L6 완료 = 대량 처리 환경에서 안정적으로 동작한다 (실패 복구 + 재개)

## Minimal Multi-Source GUI Spec (docs)

별도 설계 문서 `minimal_multi_source_gui_spec.md`에서 **최소한의 multi-source GUI**를 정의 — "source 하나만 있을 때와 여러 개일 때 UI가 어떻게 점진적으로 확장되는가"에 대한 방향 제시. P06/P07 UI Layer의 기반.

## 관련 페이지

- [PaperMeister 개요](papermeister-overview.md) — 6-layer stack 요약
- [아키텍처 & 데이터 모델](papermeister-architecture.md) — source-first UI의 이론적 배경
- [OCR 파이프라인](papermeister-ocr-pipeline.md) — L1 구현
- [LLM Biblio Extraction](papermeister-biblio-extraction.md) — L2 구현
- [Zotero Integration](papermeister-zotero-integration.md) — L5 write-back
- [제품 전략](papermeister-product-strategy.md) — P06 feature scope의 상업적 근거

---
*Sources: P01 (MVP architecture + PyQt6 layout), 001 (초기 구현), 006 (CLI 구현), 007 (CLI pagination), 009 (CLI 버그픽스 + multi-select), P06 (Desktop MVP Feature Definition), P07 (Desktop Software Implementation Plan 6 Layer), docs/minimal_multi_source_gui_spec.md.*
