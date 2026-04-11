# Trilobase 프로젝트 개요

**Trilobase**는 삼엽충(Trilobita) 분류학 데이터를 관리하는 SQLite 기반 데이터베이스 프로젝트로, [SCODA](scoda.md) 아키텍처 위에 구축되었다. 이후 brachiobase, graptobase 등 다른 화석 분류군으로 확장되어 [multi-package 생태계](packages.md)를 형성했다.

## 핵심 구성

| 구성요소 | 역할 |
|---|---|
| **trilobase DB** | 삼엽충 분류학 정전(canonical) 데이터베이스 |
| **[SCODA](scoda.md)** | Scientific Collection Open Data Architecture — 자기 기술적 데이터 패키지 규격 |
| **[scoda-engine](scoda-engine.md)** | 범용 SCODA 뷰어/서버 런타임 (별도 레포) |
| **[assertion model](assertion-model.md)** | 다중 분류 의견을 관리하는 assertion 중심 데이터 모델 |

## 기술 스택

- **Backend**: FastAPI (Flask에서 마이그레이션), SQLite, Pydantic
- **Frontend**: SPA (Bootstrap 5), D3.js (트리/차트 시각화)
- **배포**: PyInstaller 단일 EXE, .scoda 패키지, GitHub Actions CI/CD
- **AI 통합**: MCP Server (14 tools, stdio 모드, Claude Desktop 연동)
- **GUI**: tkinter 컨트롤 패널

## 데이터 규모

- 삼엽충 속(Genus): 5,113건
- 분류 계층(taxonomic_ranks): 5,338건 (Order~Genus)
- 동의어(synonyms→opinions): 1,055건
- 참고문헌(bibliography): 2,130건
- 화석산지(formations): 4,854건
- 분류 의견(taxonomic_opinions/assertions): 8,331건+ (v0.2.0)
- 분류 프로필(classification_profiles): treatise1959, treatise1997, JA2002

## 개발 타임라인

| 기간 | 주요 마일스톤 |
|---|---|
| 2026-02-04~05 | Phase 1-12: [데이터 정제](taxonomy-data.md), DB 생성, 웹 UI, 참고문헌 |
| 2026-02-07~08 | Phase 13-21: [SCODA](scoda.md) 핵심 구현, 릴리스, 로컬 오버레이, GUI |
| 2026-02-09~10 | Phase 22-25: [MCP 서버](mcp-server.md), SSE/stdio 모드 |
| 2026-02-12 | Phase 26-30: 지리/시간 데이터, ICS 지질연대표 |
| 2026-02-13 | Phase 31-43: PaleoCore DB, 범용 SCODA 뷰어, 선언적 매니페스트 |
| 2026-02-14~15 | Phase 44-49: SPA, FastAPI 마이그레이션, UID, Pydantic |
| 2026-02-16~18 | 의견 모델 설계, 매니페스트 검증, POC |
| 2026-02-19 | scoda-engine 레포 분리, DB 재구성 |
| 2026-02-20~27 | 분류학 데이터 품질 수정, 동의어→의견 마이그레이션 |
| 2026-02-28 | [rebuild 파이프라인](taxonomy-data.md), [assertion DB](assertion-model.md) |
| 2026-03-01~02 | [Radial tree](visualization.md), 프로필 셀렉터, CRUD |
| 2026-03-07~09 | Treatise 1959 임포트, [프로필 비교 시각화](visualization.md) |
| 2026-03-11 | v0.3.0 통합 (assertion DB = primary trilobase) |
| 2026-03-12~18 | [Multi-package 확장](packages.md): brachiobase, graptobase, paleobase |

## 버전 이력

| 버전 | 주요 변경 |
|---|---|
| 0.1.0 | 초기 데이터베이스 + SCODA 메타데이터 |
| 0.2.0~0.2.5 | taxon_bibliography, Agnostida, 동의어 마이그레이션, 지리 데이터 수정 |
| 0.3.0 | assertion-centric DB를 기본 trilobase로 통합, 레거시 제거 |
| 0.3.3~0.3.4 | 패키지 통합, paleobase 지원 |

## 관련 페이지

- [SCODA 아키텍처](scoda.md) — 패키지 규격 및 계층 구조
- [scoda-engine](scoda-engine.md) — 뷰어/서버 런타임
- [Assertion 모델](assertion-model.md) — 다중 분류 의견 관리
- [분류학 데이터](taxonomy-data.md) — 데이터 정제 및 rebuild 파이프라인
- [시각화](visualization.md) — 트리, 차트, 비교, 애니메이션
- [MCP 서버](mcp-server.md) — LLM 통합
- [Multi-package](packages.md) — brachiobase, graptobase, paleobase
- [Treatise 추출](treatise-extraction.md) — Treatise 원문에서 분류 체계 추출
- [PaperMeister 아키텍처](papermeister-architecture.md) — PaleoBase ↔ PaperMeister feedback loop (문헌 corpus → 도메인 DB 공급)
- [Noematica 브랜드](noematica-brand.md) — Trilobase는 Noematica 생태계의 domain knowledge 레이어

---
*Sources: raw/trilobase-devlog/ (103 files) + raw/trilobase-devlog/archive/ (~150 files), 2026-02-04 ~ 2026-03-18*
