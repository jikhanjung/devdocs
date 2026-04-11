# devdocs — 레포 전체 개요

이 레포는 **개발 문서 아카이브**다. 여러 프로젝트의 원본 devlog과 문서를 `raw/` 아래에 immutable source material로 모으고, `wiki/` 아래에 LLM으로 유지보수되는 지식 레이어를 둔다. 전체 패턴은 [llm-wiki.md](../llm-wiki.md)에 기술되어 있다.

> **개별 프로젝트의 개요는 각 프로젝트 페이지로**: [FSIS2026/KOFHIN](fsis2026-overview.md) · [Trilobase](trilobase-overview.md) · [Modan2](modan2-overview.md) · [SCODA Engine](scoda-engine.md) · [PaperMeister](papermeister-overview.md). 페이지 단위 네비게이션은 [index.md](index.md)가 authoritative.

## 포함된 프로젝트

| # | 프로젝트 | 한 줄 요약 | 기간 | raw 소스 | 개요 페이지 |
|---|---|---|---|---|---|
| 1 | **FSIS2026 / KOFHIN** | 한반도 화석산지·표본·문헌 통합 Django 웹 시스템 (국가유산청 발주) | 2026-02-24 ~ 2026-04-11+ (진행 중) | `raw/fsis2026-devlog/` (90 files) + `raw/fsis2026-docs/` (10 files) | [fsis2026-overview.md](fsis2026-overview.md) |
| 2 | **Trilobase** | 삼엽충 분류학 SQLite DB. SCODA 아키텍처 위에 구축되어 multi-package 생태계(brachiobase/graptobase/paleobase)로 확장 | 2026-02-04 ~ 2026-03-18 | `raw/trilobase-devlog/` (103 files + ~150 archive) | [trilobase-overview.md](trilobase-overview.md) |
| 3 | **Modan2** | 기하학적 형태 계측학 PyQt5 데스크톱 앱. 9일 코드 품질 현대화 스프린트 | 2025-08-28 ~ 2025-09-05 | `raw/Modan2-devlog/` (29 files) | [modan2-overview.md](modan2-overview.md) |
| 4 | **SCODA Engine** | 범용 SCODA 뷰어/서버 런타임. 2026-02-19에 Trilobase에서 독립 레포로 분리 | 2026-02-19 ~ 2026-03-18 | `raw/scoda-engine-devlog/` (97 files) | [scoda-engine.md](scoda-engine.md) |
| 5 | **PaperMeister** | Zotero+로컬폴더+EndNote를 source로 묶는 로컬 연구 코퍼스 플랫폼. [Noematica](noematica-brand.md)의 L1 제품 | 2026-03-30 ~ 2026-04-10 | `raw/PaperMeister-devlog/` (25 files) + `raw/PaperMeister-docs/` (31 files) | [papermeister-overview.md](papermeister-overview.md) |

## 프로젝트 간 관계

두 개의 독립된 계열이 공존한다:

### 1. Noematica 생태계 — 회사 브랜드 아래의 제품군

**[Noematica](noematica-brand.md)** (2026-04-03 네이밍 세션에서 확정)는 학술 인프라 / 지식 시스템 회사로, 다음 제품을 포괄한다:

- **[PaperMeister](papermeister-overview.md)** — L1 제품. Zotero 등 이종 source에서 canonical corpus를 만든다.
- **[SCODA Engine](scoda-engine.md)** — 자기 기술적 과학 데이터 패키지(`.scoda`) 뷰어/서버 런타임.
- **[Trilobase](trilobase-overview.md)** / PaleoBase — SCODA 위에 구축된 도메인 지식 패키지. SCODA Engine의 첫 consumer.

**데이터 흐름**: PaperMeister (corpus 수집/OCR/추출) → SCODA domain 패키지 (trilobase/paleobase) → scoda-engine (뷰어/서버). 즉 PaperMeister가 upstream, SCODA/Trilobase가 downstream인 파이프라인이며, PaleoBase feedback loop로 도메인 지식이 PaperMeister의 추출 정확도를 향상시키는 순환 구조.

### 2. 독립 프로젝트

- **[FSIS2026 / KOFHIN](fsis2026-overview.md)** — 국가유산청 용역 사업이라 계약상 분리된 Django 프로덕션 시스템. 다만 PDF/OCR 파이프라인과 참고문헌 처리는 PaperMeister와 기법적 overlap이 크고, 북한 논문 corpus 처리는 PaperMeister가 일반 플랫폼으로 해결하려는 문제의 도메인 사례로 볼 수 있다.
- **[Modan2](modan2-overview.md)** — 기하학적 형태 계측학 독립 GUI 앱. 다른 프로젝트와 직접적인 코드 공유는 없지만 고생물학 워크플로우에서 쓰이는 도구 계열.

## 레포 구조

```
devdocs/
├── raw/                         # immutable source material (절대 수정 금지)
│   ├── fsis2026-devlog/
│   ├── fsis2026-docs/
│   ├── trilobase-devlog/
│   ├── Modan2-devlog/
│   ├── scoda-engine-devlog/
│   ├── PaperMeister-devlog/
│   └── PaperMeister-docs/
├── wiki/                        # 유지보수되는 지식 레이어
│   ├── index.md                 # authoritative navigation
│   ├── overview.md              # 이 파일 — 레포 전체 meta-overview
│   ├── log.md                   # ingest/lint 작업 기록
│   ├── source-status.md         # raw/ 인제스트 처리 현황
│   └── {project}-overview.md + 엔티티/개념 페이지들
├── graphify-out/                # /graphify 출력 (지식 그래프 artifacts)
├── llm-wiki.md                  # LLM Wiki 패턴 참고 문서
├── CLAUDE.md                    # repo-specific Claude Code rules
└── .obsidian/                   # Obsidian 설정 (obsidian-git plugin으로 동기화)
```

## Devlog 파일 네이밍 규약

`raw/*-devlog/` 파일은 `YYYYMMDD_NNN_description.md` 패턴을 따른다:

- 숫자 prefix (`001`, `078`) = 구현/세션 로그
- `P` prefix (`P01`, `P62`) = 계획(planning) 문서
- `R` prefix (`R01`, `R03`) = 회고(review) 문서

## 주요 워크플로우

- **지식 레이어 구축**: `raw/`의 새 source는 [llm-wiki.md](../llm-wiki.md) 패턴에 따라 ingest → wiki 페이지 업데이트 → [index.md](index.md) / [log.md](log.md) 갱신
- **그래프 시각화**: `/graphify` skill은 raw devlog을 클러스터링된 커뮤니티와 HTML/JSON 시각화로 변환한다 → `graphify-out/`
- **뷰어**: 레포는 Claude Code로 편집하고 Obsidian(obsidian-git plugin으로 동기화)으로 탐색한다. `.obsidian/`은 git 트래킹되며 휘발성 local state만 `.gitignore`에서 제외됨.

## 메타 페이지

- [index.md](index.md) — 모든 wiki 페이지의 authoritative 목차
- [source-status.md](source-status.md) — raw/ 디렉토리별 ingest 처리 현황
- [log.md](log.md) — ingest 및 lint 작업 기록

---
*Sources: [CLAUDE.md](../CLAUDE.md), [llm-wiki.md](../llm-wiki.md), 각 프로젝트 개요 페이지 + [noematica-brand.md](noematica-brand.md) (회사 브랜드 관계 정보).*
