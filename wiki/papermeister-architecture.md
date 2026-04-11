# PaperMeister Sync-Centric 아키텍처 & 데이터 모델

PaperMeister의 핵심 아키텍처 결정. **"External systems are source systems; PaperMeister is the canonical corpus system"**이라는 원칙 아래, 여러 외부 문헌 도구(Zotero, EndNote, 디렉토리)의 상위에 앉아 이들을 동기화·병합·정규화하여 하나의 *canonical paper corpus*를 유지한다. 2026-04-03의 설계 스프린트(docs/sync_centric_architecture_spec, data_model_revision_spec)에서 현재 형태로 확정되었다.

## 1. 제품 포지션

PaperMeister는 **아래 도구들을 대체하지 않는다**:
- Zotero
- EndNote
- 디렉토리 기반 PDF 아카이브
- 미래의 RIS / BibTeX / Mendeley / watched folder 등

이들은 **source system**으로 남고, PaperMeister는 그 위에서:

1. 이종 source 수집 (ingest)
2. 레코드 reconcile · merge
3. OCR 및 텍스트 추출
4. 구조화 corpus 생성
5. 분석 + assertion 레이어
6. 질문 기반 retrieval / summarization (장기)

를 담당하는 **canonical corpus system**이다.

## 2. 아키텍처 원칙

> **External systems are source systems; PaperMeister is the canonical corpus system**

의미:
- 외부 시스템은 자기 identifier와 lifecycle을 유지한다
- PaperMeister는 **sync snapshot**과 **canonical 내부 레코드**를 별도로 보관한다
- OCR과 derived knowledge는 PaperMeister의 소유다
- **source sync와 corpus update는 first-class feature**다

## 3. Source 카테고리

PaperMeister는 세 종류의 source를 구분한다:

| 카테고리 | 예시 | 특성 |
|---|---|---|
| **Bibliographic** | EndNote, RIS, BibTeX | 메타데이터 풍부, 파일은 부재/약한 연결 |
| **File** | 로컬 디렉토리, 네트워크 폴더, watched archive | PDF 풍부, 메타데이터 빈약·불균일 |
| **Hybrid** | Zotero | 메타데이터 + 첨부 파일 graph + collection hierarchy + 안정 identifier |

한 사용자가 **여러 source를 동시에 등록 가능**해야 한다. 예:
- Zotero = 활성 라이브러리
- 로컬 폴더 = 과거 PDF 아카이브
- EndNote export = 오래된 bibliography

전역으로 하나의 source model을 선택하도록 강요하지 않는다. 여러 source가 공존하며 **하나의 canonical corpus**에 기여하고, source-specific provenance는 UI에 계속 표시된다.

## 4. 3-Layer Canonical Record 전략

데이터 모델은 **세 레이어**를 명확히 분리한다:

```
┌─────────────────────────────────────────────────┐
│ Layer 1: External Source Snapshot               │
│   - Zotero item snapshot                        │
│   - EndNote record snapshot                     │
│   - Directory file discovery snapshot           │
│   → 원본 identifier와 source state 보존          │
│   → sync 전/후 비교, change detection 지원       │
└──────────────────┬──────────────────────────────┘
                   │ maps to (N:1)
                   ▼
┌─────────────────────────────────────────────────┐
│ Layer 2: Canonical Paper Entity                 │
│   - 내부 PaperMeister paper                      │
│   - "이 source observation들은 동일 논문인가?"    │
│   - 여러 source record가 하나의 canonical에 merge │
└──────────────────┬──────────────────────────────┘
                   │ owns
                   ▼
┌─────────────────────────────────────────────────┐
│ Layer 3: Derived Corpus Data                    │
│   - OCR outputs                                 │
│   - Passages, structured text                   │
│   - Future: assertions, entities                │
│   → canonical Paper/PaperFile에 귀속            │
└─────────────────────────────────────────────────┘
```

**원칙**:
- Zotero item, EndNote record, 디렉토리 PDF는 **source observation**이다 — 이들은 canonical paper가 **아니다**
- 여러 observation이 **하나의 canonical Paper에 매핑**될 수 있다
- OCR output, passages, assertions는 **canonical entity에 부착**되며 raw source record에 직접 묶이지 않는다
- Provenance는 항상 보존: "어떤 paper가 어디서 왔고, 어떤 external record가 이를 기술했으며, 어떤 external file이 PDF를 공급했는지"

## 5. 현재 데이터 모델 (MVP)

```
Source
  ├── Folder (filesystem dir or Zotero collection)
  │     └── Paper
  │           ├── Author (M:N)
  │           ├── PaperFile   (zotero_key, hash, status)
  │           └── Passage     (page, text → FTS5 index)
```

**작동했던 이유**: Zotero 중심 MVP에서는 source-specific 정보가 어느 정도 canonical entity에 스며들어도 괜찮았다.

**한계**:
- `Paper.zotero_key`, `PaperFile.zotero_key` → source 정보가 canonical entity에 leak
- EndNote-류 bibliographic-only source 지원 어려움
- 디렉토리-only file source 지원 어려움
- 여러 source가 같은 paper를 기술할 때 처리 불명확
- Sync snapshot tracking 부재
- Source record lifecycle 관리 부재

## 6. 개정 데이터 모델 (제안, 2026-04-03)

### 6.1 `Source`

한 사용자가 등록한 하나의 외부 source system.

**예**: Zotero library 1개, EndNote import 1개, 로컬 디렉토리 1개, watched archive 1개

**필드**:
- `id`, `name`
- `source_type`: `zotero` | `endnote` | `directory` | `ris` | `bibtex`
- `source_class`: `hybrid` | `bibliographic` | `file`
- `path` 또는 동등한 locator
- `config_json`
- `sync_status`, `last_synced_at`
- `created_at`, `updated_at`

현재 `Source` 테이블을 **교체하지 않고 진화**시킨다.

### 6.2 `Folder` (Source-Specific Hierarchy Node)

현재 `Folder` 추상화는 유지 — source별 hierarchy 컨테이너로 여전히 유용:
- filesystem 디렉토리
- Zotero collection
- 미래 source-specific group node

Canonical paper에는 직접 연결되지 않고 **source observation layer에 속한다**.

### 6.3 External Source Snapshot (신규 개념)

각 source에서 관찰된 레코드를 **그대로 보관**. Zotero item snapshot, EndNote record snapshot, 디렉토리 파일 발견 snapshot 등. Sync 시 이전 상태와 비교하여 변경 탐지.

### 6.4 Canonical `Paper` + `PaperFile`

내부 canonical entity. External identifier를 갖지 않는다(혹은 보조 필드로만 유지). OCR output, passages, 파생 데이터는 모두 여기에 부착.

### 6.5 파생 corpus layer

Passage, structured text cache, 향후 entity/assertion 모두 canonical `Paper`/`PaperFile`에 부착. Source record에 직접 의존하지 않음 → source가 re-sync 되거나 삭제되어도 corpus는 안정.

## 7. 현재 MVP와 개정안 사이

개정 모델은 **점진적 진화**다. 이미 MVP가 동작하고 있으므로 `Paper.zotero_key` 같은 필드를 당장 제거하지 않고, 점차 source snapshot layer로 이주시킨다. `Folder`도 유지하되 의미만 재정의(source hierarchy node).

→ 구체적 migration 전략은 devlog 008 (Paper Zotero key + standalone PDF), 013 (resync + hash matching) 참조.

## 8. SCODA로 이어지는 6-Layer Pipeline

PaperMeister corpus는 곧바로 SCODA 패키지가 되지 **않는다**. 그 사이에 **domain knowledge construction 단계**가 필요하다.

| Layer | 소유자 | 내용 |
|---|---|---|
| **L1. Source Layer** | external | Zotero, EndNote, 로컬 PDF, scanned archive |
| **L2. PaperMeister Corpus Layer** | PaperMeister | source ingest, sync, OCR, raw OCR JSON, searchable passages, 구조화 corpus 후보 |
| **L3. Domain Extraction Layer** | PaperMeister (수동 + LLM) | entity candidate extraction, relation/assertion extraction, bibliography linking, evidence bundling |
| **L4. Canonical Domain DB Layer** | domain-specific | normalized taxa, formations/locations/times, synonym relations, bibliographic references, curated assertions |
| **L5. SCODA Package Layer** | SCODA spec | SQLite DB, provenance, named queries, UI manifest, entity schema, dependency metadata |
| **L6. SCODA Runtime / Distribution Layer** | [SCODA Engine](scoda-engine.md) | generic viewer, MCP access, web/desktop runtime, [Hub distribution](scoda-hub.md) |

### PaperMeister의 역할

PaperMeister는 **L2**를 담당한다:
- 문헌 수집, source provenance 유지
- OCR + layout 정보 확보
- 전문 검색, 구조 보존
- 지속적 corpus 갱신

즉 PaperMeister는 "읽을 수 있는 문헌 corpus"를 만든다. 가치는 크지만, 아직 SCODA 배포 artifact는 **아니다** — 여전히 문헌 중심 / assertion 미정규화 상태.

### 중간 단계 — Domain Extraction (L3)

PaperMeister → SCODA 사이에 가장 중요한 중간 단계. 이 단계에서:
- 어떤 엔티티를 추출할지 **도메인별**로 정의 (taxon, formation, locality, ...)
- 어떤 관계를 assertion으로 볼지 정의
- OCR/텍스트에서 후보를 뽑기
- bibliographic evidence를 assertion에 연결

결과물이 충분히 정규화되면 **L4 canonical domain DB**로, 거기서 **L5 SCODA 패키지**로 변환되어 [SCODA Hub](scoda-hub.md)를 통해 배포된다.

## 9. PaleoBase Feedback Loop

PaperMeister와 [PaleoBase](packages.md) 계열(trilobase, brachiobase, graptobase 등) 사이에는 **양방향 피드백 루프**가 설계되어 있다:

- **PaperMeister → PaleoBase**: 새 논문의 OCR/extraction 결과가 PaleoBase assertion 후보로 공급
- **PaleoBase → PaperMeister**: 확정된 canonical taxon/locality가 PaperMeister의 entity normalization 사전으로 역류, 검색/추출 품질 향상

이 루프는 PaperMeister를 **단일 제품**이 아니라 **Noematica 생태계의 문헌 infrastructure**로 자리잡게 한다.

→ 상세: docs/papermeister_paleobase_feedback_loop.md

## 관련 페이지

- [PaperMeister 개요](papermeister-overview.md)
- [OCR 파이프라인](papermeister-ocr-pipeline.md) — L2의 OCR 부분
- [LLM Bibliographic Extraction](papermeister-biblio-extraction.md) — L3의 bibliographic 부분
- [Zotero Integration](papermeister-zotero-integration.md) — L1 hybrid source 구현
- [SCODA 아키텍처](scoda.md) — L5 패키지 규격
- [SCODA Engine](scoda-engine.md) — L6 런타임
- [SCODA Hub](scoda-hub.md) — L6 배포
- [Trilobase Packages](packages.md) — L4 domain DB 예시 (paleobase, brachiobase 등)
- [Noematica 브랜드](noematica-brand.md) — PaperMeister/SCODA/PaleoBase 우산 브랜드

---
*Sources: docs/sync_centric_architecture_spec.md, docs/data_model_revision_spec.md, docs/papermeister_to_scoda_pipeline.md, docs/papermeister_paleobase_feedback_loop.md, devlog P01, 008, 013.*
