# PaperMeister Zotero Integration

Zotero는 PaperMeister의 가장 초기 구현 대상이자 현재 가장 성숙한 hybrid source이다. 2026-03-30의 계획(P02)부터 04-09의 standalone promote + vision pass(016)까지, Zotero lifecycle 전반 — sync, pagination, standalone PDF 처리, resync, hash matching, OCR JSON write-back — 이 점진적으로 구축되었다.

Zotero는 [source 카테고리](papermeister-architecture.md#3-source-카테고리) 중 **hybrid** (메타데이터 + 첨부파일 graph + collection hierarchy) 에 해당하며, `~9,500`편의 parent item과 `~50`편의 standalone PDF를 제공한다.

## 지원 범위 개요

| 기능 | devlog | 상태 |
|---|---|---|
| Zotero API 연동 + collection sync | P02, 003 | ✅ |
| Parent item 기반 논문 import | 003 | ✅ |
| Collection pagination (대량 데이터) | 007 | ✅ |
| Paper에 `zotero_key` 부착 | 008 | ✅ (현재 MVP, 장기적으로 source snapshot으로 이주 예정) |
| Standalone PDF 처리 (parent 없는 첨부) | 008 | ✅ 기본 |
| Zotero resync + hash matching | 013 | ✅ |
| OCR JSON → Zotero 첨부 업로드 | 014 | ✅ |
| Standalone PDF → parent promote | 016 | 🚧 review queue 기반 |
| Vision pass (이미지 기반 서지정보 추출) | 016 | 🚧 |
| Zotero write-back (LLM biblio 결과) | P07 L5 | ⏳ 계획 |

## 초기 Integration (P02 → 003, 2026-03-30 ~ 03-31)

### P02 계획의 핵심 질문

- Zotero API 접근 방식: web API vs. 로컬 SQLite 직접
- 인증: user API key
- 수집 대상: 전체 라이브러리 vs. 특정 collection
- PDF 첨부: 로컬 파일 경로 vs. API 다운로드

### 003 구현 결정

- **Zotero Web API** 사용 (pyzotero 또는 직접 HTTP)
- **Collection 단위 sync** — 사용자가 컬렉션 선택 후 import
- **Parent item + 첨부 graph** 탐색
- 각 Zotero item → PaperMeister `Paper` 1:1 매핑 (MVP)
- PDF 첨부 경로는 Zotero의 로컬 저장소에서 읽기

이 시점의 데이터 흐름:

```
Zotero Collection
    ↓ (parent items)
Zotero Items (top-level, with metadata)
    ↓ (child attachments)
PDF files (로컬 Zotero storage)
    ↓ (hash)
PaperMeister Paper + PaperFile
    ↓ (OCR → passages → FTS5)
searchable corpus
```

## Collection Pagination (007, 2026-04-01)

대형 collection (~수천 편)을 한 번에 가져오면:
- Zotero API rate limit 초과
- UI block
- 실패 시 전체 재시작

### 해결

- **페이지 단위 iteration**: `start` + `limit` 파라미터로 100편씩
- 각 페이지 받을 때마다 **점진적으로 DB commit** → crash 복구 가능
- UI는 progress bar + 취소 가능
- 중간 취소 후 재개 시 이미 import된 item은 skip (Zotero key 기반)

## `zotero_key` 필드와 Canonical Merge 긴장 (008)

### 문제

`Paper.zotero_key`, `PaperFile.zotero_key`를 MVP 편의상 canonical entity에 직접 붙여두었다. 동작은 하지만 [sync-centric 아키텍처](papermeister-architecture.md) 원칙에 어긋난다 — source-specific 식별자는 source snapshot layer에 있어야지 canonical layer에 있으면 안 된다.

### 008의 처리

단기적 타협:
- `zotero_key`는 일단 유지 (MVP 작동 유지)
- **Standalone PDF** (parent 없는 Zotero 첨부) 처리 추가 — 이 경우 parent metadata가 없으므로 Paper는 제목/저자 없이 생성되고, LLM biblio extraction이 나중에 채움
- Standalone PDF는 `parent_key == null`로 식별

장기적으로는 데이터 모델 개정(data_model_revision_spec)에 따라 `zotero_key`를 source snapshot 테이블로 이주 예정.

## CLI 버그픽스 + Multi-Select (009, 2026-04-02)

- CLI에서 여러 collection을 한 번에 sync할 수 있도록 `--collection` 인수를 복수 지정 허용
- Collection ID 목록 파싱 버그 수정
- 부분 실패 시에도 나머지 collection은 계속 진행

## Zotero Resync + Hash Matching (013, 2026-04-03)

### 문제

사용자가 Zotero에서 item을 업데이트하거나 파일을 교체한 경우, PaperMeister가 이를 탐지해야 함. 단순히 `zotero_key`로만 매칭하면 파일이 바뀌어도 재처리 안 됨.

### 해결 — 2단계 매칭

1. **Zotero key 매칭** — 기존 item이면 업데이트 대상으로 표시
2. **PDF hash 매칭** — 첨부 파일의 SHA-256 해시를 비교
   - 해시 동일 → 메타데이터만 업데이트 (OCR 재실행 불필요)
   - 해시 상이 → 새 `PaperFile` 등록, OCR 재실행 필요

이 설계는 [OCR JSON 캐시](papermeister-ocr-pipeline.md)가 **hash 기반**인 점과 자연스럽게 맞물린다 — 같은 파일이면 이미 캐시된 OCR JSON을 재사용.

## OCR JSON → Zotero Upload (014, 2026-04-07)

### 동기

OCR 결과(raw JSON)를 PaperMeister 내부에만 두면 Zotero 사용자는 볼 수 없다. Zotero로 돌아갔을 때도 검색/탐색이 가능하도록 **OCR 결과를 Zotero 첨부로 업로드**.

### 구현

- OCR JSON을 Zotero item의 **child attachment**로 업로드
- Note type (JSON 내용을 요약한 Markdown) + File type (raw JSON) 두 가지 고려
- 업로드는 **사용자 선택** (자동 업로드는 Zotero storage 부담 주므로 opt-in)
- 업로드 실패 시에도 PaperMeister 내부 OCR 상태는 유지

## Standalone Promote + Vision Pass (016, 2026-04-09)

### 문제

Zotero standalone PDF (~50편) — parent item 없이 첨부만 존재:
- 메타데이터 없음 (제목조차 파일명에서 유추)
- PaperMeister가 이들을 검색 가능하게 만들어도, Zotero UI에서는 여전히 "untitled attachment"로 남음
- 장기적으로 이 파일들을 **parent item으로 승격**시켜야 함

### 해결 — Standalone Promote Flow

```
Standalone PDF
    ↓ (OCR)
OCR JSON + raw text
    ↓ (LLM biblio extraction)   ← papermeister-biblio-extraction.md
PaperBiblio (extracted)
    ↓ (confidence check)
    ├── high  → Auto-promote to parent item (Zotero write-back)
    ├── medium → Review queue (user 승인 후 promote)
    └── low   → Vision pass 필요 → review queue
```

### Vision Pass

- 텍스트 기반 추출의 confidence가 낮으면 **page 이미지 자체를 vision LLM에 전달**
- 비용이 비싸므로 standalone + low confidence 케이스에만 발동
- 결과는 같은 `PaperBiblio` 스키마 (`source: "llm_vision"`)

→ 상세: [biblio extraction의 Vision Pass 섹션](papermeister-biblio-extraction.md#vision-pass-016)

## Zotero Write-back 정책 (P07 Layer 5)

[P07 Desktop 구현 계획](papermeister-cli-and-gui.md)에서 Zotero write-back은 **Layer 5 Controlled Automation**에 배치된다:

- 자동 실행되지 않음 (사용자 승인 필요)
- Review queue를 통해 명시적으로 promote
- Write-back 실패 시 rollback 가능해야 함
- 로그 + audit trail 유지

**원칙**: PaperMeister가 외부 source를 *subtle하게* 오염시키지 않는다. 모든 write-back은 사용자가 의도한 결과여야 함.

## Source Provenance 표시

UI와 CLI에서 각 Paper가 어디서 왔는지 항상 표시:
- `source: zotero` + collection 경로
- `source: directory` + 파일 경로
- `source: endnote` + import file
- Hybrid (여러 source가 같은 paper를 기술) → 다중 표시

이는 [canonical paper + source snapshot 분리](papermeister-architecture.md#4-3-layer-canonical-record-전략) 원칙의 UI 반영.

## 관련 페이지

- [PaperMeister 개요](papermeister-overview.md)
- [아키텍처 & 데이터 모델](papermeister-architecture.md) — source 카테고리 / canonical merge
- [OCR 파이프라인](papermeister-ocr-pipeline.md) — hash 기반 캐싱이 resync 전략의 전제
- [LLM Biblio Extraction](papermeister-biblio-extraction.md) — Zotero 9,500편이 ground truth
- [CLI & GUI](papermeister-cli-and-gui.md) — Zotero sync의 사용자 경로
- [제품 전략](papermeister-product-strategy.md) — Zotero 사용자가 첫 번째 target 고객

---
*Sources: P02 (Zotero integration plan), 003 (구현), 007 (CLI collection pagination), 008 (Paper Zotero key + standalone), 009 (CLI 버그픽스 + multi-select), 013 (resync + hash matching), 014 (OCR JSON Zotero upload), 016 (standalone promote + vision pass).*
