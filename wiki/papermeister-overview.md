# PaperMeister 프로젝트 개요

**PaperMeister**는 Zotero, 로컬 폴더, EndNote 등 이종 소스에서 논문을 수집·OCR·전문 검색하는 **로컬 연구 코퍼스 플랫폼**이다. 단순 PDF 뷰어나 reference manager가 아니라, 외부 도구들을 source system으로 두고 그 위에 **canonical corpus layer**를 구축하는 아키텍처로 설계되었다. 장기적으로는 [SCODA/PaleoBase 도메인 패키지](papermeister-architecture.md)로 이어지는 파이프라인의 출발점이 된다.

이 프로젝트는 [Noematica](noematica-brand.md)의 두 제품 중 하나다 (다른 하나는 [SCODA Engine](scoda-engine.md)).

## 핵심 원칙: Store first, understand later

- raw OCR output은 손실 없이 보존
- derived layer는 재현 가능해야 함 (원본으로부터 다시 생성 가능)
- interpretation은 additive — 원본을 파괴하지 않음
- 고급 지능은 **안정된 데이터 자산 위에** 쌓는다. 기능 폭이 아니라 데이터 모델 깊이가 우선.

## 기술 스택

| 항목 | 선택 | 이유 |
|---|---|---|
| 언어 | Python 3.10+ | 과학 문서 처리 생태계 |
| GUI | PyQt6 | 크로스플랫폼 데스크톱 |
| DB | SQLite + FTS5 | 단일 파일, BM25 전문 검색 내장 |
| ORM | Peewee | 경량, SQLite FTS5 확장 지원 |
| PDF 텍스트 추출 | PyMuPDF (`fitz`) | 텍스트 + 메타데이터 + 페이지 구조 |
| OCR 엔진 | **RunPod Chandra2** | GPU 기반, section/caption/body/layout 구조 인식 |
| 대안 OCR 실험 | Ollama GLM-OCR | 오프라인 / 로컬 가능성 (005 검토) |
| LLM biblio 추출 | Claude Haiku/Sonnet, GPT-4o-mini | 서지정보 구조화 (→ [biblio extraction](papermeister-biblio-extraction.md)) |

## 현재 MVP 상태 (2026-04-10 기준)

| 영역 | 구현 상태 |
|---|---|
| 로컬 폴더 스캔 + PDF ingest | ✅ |
| 해시 기반 dedup | ✅ |
| Zotero 컬렉션 sync + 논문 import | ✅ (→ [Zotero integration](papermeister-zotero-integration.md)) |
| RunPod Chandra2 OCR | ✅ |
| Raw OCR JSON 캐싱 (`~/.papermeister/ocr_json/{hash}.json`) | ✅ |
| SQLite + FTS5 passage 검색 | ✅ (BM25, snippet) |
| PyQt6 GUI (메인 윈도우 + 검색 + 상세) | ✅ |
| CLI (import, processing, search, listing, zotero ops) | ✅ (→ [CLI & GUI](papermeister-cli-and-gui.md)) |
| 병렬 OCR | ✅ (004, P03) |
| Batch retry / resume / concurrency | ✅ (018) |
| LLM bibliographic extraction | 🚧 실험 → 본격 도입 단계 (P04, 015, 012) |
| Standalone PDF promote + vision pass | 🚧 (016) |
| 구조화 corpus 레이어 (Phase 2) | ⏳ 계획 중 |

## 12일 devlog 타임라인 (2026-03-30 ~ 2026-04-10)

| 날짜 | 파일 | 주요 작업 |
|---|---|---|
| **03-30** | P01, 001 | MVP 아키텍처 + 초기 구현 (models, DB, FTS5, PyQt6 UI) |
| 03-30 | 002 | OCR 파이프라인 + 처리 UI |
| 03-30 | P02 | Zotero integration 계획 |
| **03-31** | 003 | Zotero integration 구현 |
| 03-31 | 004, P03 | 병렬 OCR 처리 |
| 03-31 | 005 | Ollama GLM-OCR 테스트 |
| **04-01** | 006, 007 | CLI 구현 + Collection pagination |
| **04-02** | 008, 009 | Paper Zotero key + standalone PDF, CLI 버그픽스 + multi-select |
| **04-03** | **010** | **프로젝트 Overview + 다음 방향 (pivot point)** — 현재 5-phase 로드맵이 이 세션에서 정리됨 |
| 04-03 | 013 | Zotero resync + hash matching |
| **04-07** | 014 | OCR JSON Zotero upload |
| **04-08** | 011, 015, P04, P05 | LLM 모델 비교 + biblio pipeline + 추출 계획 |
| **04-09** | 012, 016, 017, 018, **P06** | Biblio 교훈, standalone promote + vision pass, OCR cost (reserved GPU), batch retry/resume/concurrency, **Desktop MVP feature 정의** |
| **04-10** | **P07** | **Desktop Software 구현 계획 (6 layer)** — 가장 최신 |

4월 3일의 **010** 세션이 pivot point: 초기 "PDF 관리 앱"에서 **"구조화 연구 코퍼스 플랫폼"**으로 방향 정리. 이후 모든 작업이 그 방향에 맞춰 정렬된다.

## 장기 5-Phase 로드맵 ([specification](#) 기준)

| Phase | 목표 | 핵심 산출물 |
|---|---|---|
| **Phase 1. Foundation Stabilization** (현재) | 기존 MVP를 운영 가능한 기반으로 다듬기 | 테스트 커버리지, OCR 예외 처리, 검색 하이라이트, 대량 처리 안정성, reindex/reprocess 검증 |
| **Phase 2. Structured Corpus Layer** | OCR output을 평평한 텍스트가 아닌 재사용 가능한 구조화 코퍼스로 전환 | section/caption/body 구분, 블록/섹션 level provenance, 구조화 파생 레코드 |
| **Phase 3. Analysis Layer** | 코퍼스 탐색/분석 도구 | word freq, n-gram, TF-IDF, 키워드 비교, trend 요약. 확장: taxon/time/locality 빈도, co-occurrence, 저자 네트워크 |
| **Phase 4. Entity & Assertion Layer** | 느슨한 키워드가 아닌 검증된 assertion 형태의 도메인 지식 추출 | 대상 엔티티: taxon, formation, locality, stratigraphy, specimen, paper |
| **Phase 5. Research Assistant** | 위의 모든 레이어 위에 쌓는 도메인 인지 agent 워크플로우 | 최종 layer. agent는 **마지막** 단계 |

**순서의 중요성**: agent는 마지막이다. 좋은 corpus와 좋은 도구가 먼저 있어야 한다. → [product strategy](papermeister-product-strategy.md) 참조.

## 6-Layer Desktop 구현 스택 (P07, 2026-04-10 최신 계획)

Phase 1 내에서 데스크톱 앱을 만드는 구체적 계층:

| Layer | 내용 |
|---|---|
| **L1. Corpus Foundation** | source 등록, 스캔·동기화, PDF hash, OCR 실행, OCR JSON 캐시, 처리 상태 추적 |
| **L2. Bibliographic Layer** | OCR JSON 기반 서지정보 추출, `PaperBiblio` 저장, confidence/doc_type/needs_visual_review 처리, Paper 반영 규칙 |
| **L3. UI Layer** | source-first navigation, operational views, paper list, detail panel, preferences, process window |
| **L4. Search Layer** | FTS5 BM25, metadata + text 혼합 검색, filters/sorting/snippet |
| **L5. Controlled Automation** | review queue, vision pass, standalone promote, Zotero write-back |
| **L6. Hardening** | retry/resume/concurrency, failure taxonomy, backend switching, 암호화 PDF |

**구현 전략**:
1. **Foundation first** — L1 네 가지 (source, OCR 캐시, 상태 모델, DB 재구성 가능성)
2. **Bibliographic usability early** — 서지정보 추출을 뒤로 미루지 않음. 검색 가능 corpus의 실질 가치가 여기서 올라감.
3. **UI는 source-first** — canonical merge 성숙 전까지 source별 구조와 operational view 병행 제공
4. **자동화는 단계적으로** — 추출 저장 먼저, promote/write-back/vision은 통제 가능한 흐름으로 뒤에 붙임

## 관련 페이지

- [Sync-centric 아키텍처 & 데이터 모델](papermeister-architecture.md)
- [OCR 파이프라인 + 비용 최적화](papermeister-ocr-pipeline.md)
- [LLM Bibliographic Extraction](papermeister-biblio-extraction.md)
- [Zotero Integration](papermeister-zotero-integration.md)
- [CLI & GUI (Desktop)](papermeister-cli-and-gui.md)
- [제품 전략, MVP, GTM, vs RAG](papermeister-product-strategy.md)
- [Noematica 브랜드 & Naming](noematica-brand.md)
- [SCODA Engine](scoda-engine.md) — PaperMeister의 downstream runtime
- [Trilobase 개요](trilobase-overview.md) — PaleoBase feedback loop의 한 축

---
*Sources: raw/PaperMeister-devlog/ 25 files (2026-03-30 ~ 2026-04-10) + raw/PaperMeister-docs/ 31 files. 샘플: P01 (MVP arch), 010 (pivot), P06 (Desktop features), P07 (6-layer impl), papermeister_specification.md, sync_centric_architecture_spec.md, data_model_revision_spec.md, sellable_mvp_definition.md.*
