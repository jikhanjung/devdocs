# Wiki Index

## Repo Overview
- [devdocs 레포 전체 개요](overview.md) — 이 레포가 담는 다섯 프로젝트(FSIS2026/KOFHIN, Trilobase, Modan2, SCODA Engine, PaperMeister)와 그들 사이의 관계 (Noematica 생태계 + 독립 프로젝트 계열), 레포 구조, 워크플로우

---

## FSIS2026 / KOFHIN

> **공식명**: KOFHIN (Korea Fossil Heritage Inventory) · **프로덕션 URL**: https://fsis.psok.or.kr · 국가유산청 발주, 한국고생물학회 수행 (2026.03~2026.11, 8개월). 한반도 화석산지·표본·문헌 통합 관리 시스템.

### Project Overview
- [FSIS2026 / KOFHIN 프로젝트 개요](fsis2026-overview.md) — 사업 맥락, 4대 목표, ID 체계(GeoSite/Specimen/Reference), 5단계 수행계획, 기술 스택, 타임라인, 해외 플랫폼 벤치마크

### Entity Pages (앱/시스템)
- [KPRDB](kprdb.md) — 한국 고생물학 논문 데이터베이스 앱 (Reference-ID 소유, Author, Journal)
- [GHDB](ghdb.md) — 지질유산 데이터베이스 앱 (별도 컨테이너, Specimen-ID 연결)
- [GeoHeritage Evaluator](evaluator.md) — 화석산지 위험도 평가 (Brilha 5항목 + P0~P4 매트릭스)

### Concept Pages (주요 시스템)
- [PDF 처리 파이프라인](pdf-pipeline.md) — OCR → 정규식 추출 → Claude AI 보강, `.pdf.extract.claude.json` 우선순위 fallback
- [작업 관리 시스템](task-management.md) — 교수-학생 논문 처리 작업 할당/추적, 3 Phase 워크플로우
- [지도 시스템](map-system.md) — Kakao Maps → Leaflet → MapLibre GL JS 전환 이력
- [배포 인프라](deployment.md) — GCP 서버, Docker + Nginx, 호스트 측 cron/venv, umask 002, 환경변수 3종
- [데이터 수집](data-import.md) — jpaleodb, DBpia, CrossRef, 북한논문, Import 보고서 집계(1,354/756), 저널 dedup
- [북한 논문 데이터](north-korea-data.md) — Oh et al. 2023 seed, 101건 등록, 저자 한글 매핑 38/51, PDF 96건
- [KOFHIN 매뉴얼](kofhin-manuals.md) — 관리자 + 일반사용자 운영/입력 매뉴얼
- [UI 디자인 시스템](ui-design.md) — Earth-tone 테마, Bootstrap 5, 컴포넌트, 네비게이션
- [Django 마이그레이션](django-migration.md) — nkfadmin(Django 3.1) → fsis2026(Django 5.2) 전환 과정

---

## Trilobase

### Project Overview
- [Trilobase 프로젝트 개요](trilobase-overview.md) — 삼엽충 분류학 DB, 기술 스택, 타임라인, 버전 이력

### Data
- [분류학 데이터](taxonomy-data.md) — Phase 1-12 데이터 정제, rebuild 파이프라인, 품질 수정
- [Treatise 추출](treatise-extraction.md) — Treatise 1959/1997 OCR 추출, R04 TXT 형식, TSF 대규모 추출

### Architecture
- [Assertion 모델](assertion-model.md) — 다중 분류 의견 관리, classification profiles, edge cache

### Features
- [Multi-package](packages.md) — brachiobase, graptobase, paleobase, paleocore

---

## Modan2

> 기하학적 형태 계측학(geometric morphometrics) PyQt5 데스크톱 앱. 2025-08-28 ~ 2025-09-05 (9일) 코드 품질 현대화 스프린트 기록.

### Overview
- [Modan2 프로젝트 개요](modan2-overview.md) — 기술 스택, 타임라인, 리팩토링 전/후 지표, 파일 구조

### Architecture & Testing
- [아키텍처 & MVC 리팩토링](modan2-architecture.md) — Modan2.py 모놀리스 → main/Controller/AppSetup/Constants/Helpers 분리, 시그널-슬롯 패턴
- [테스트 인프라](modan2-testing.md) — 0 → 192 tests, 4-레벨 의존 계층, pytest-qt, xvfb CI, QMessageBox 전역 억제

### Features & Infrastructure
- [통계 분석 시스템 (PCA/CVA/MANOVA)](modan2-analysis.md) — 통합 실행, 3D 차원 처리, 4 MANOVA 통계량, 023 디버깅 세션 8 bugs
- [운영 인프라 (설정/로깅/버전/빌드/CI)](modan2-infrastructure.md) — QSettings→JSON, print→logging, version.py, Anaconda 호환, GitHub Actions
- [안정성 & 호환성](modan2-stability.md) — 전역 try/except, OpenGL/GLUT, matplotlib, xvfb, WSL 썸네일 known issue

---

## SCODA Engine

> 2026-02-19에 trilobase에서 독립 저장소로 분리된 범용 SCODA 뷰어/서버 런타임. 25일간(02-19 ~ 03-18) 집중 개발로 Desktop v0.1.x → v0.3.4+ 진화.

### Architecture
- [scoda-engine](scoda-engine.md) — FastAPI 런타임, Desktop GUI, 모노레포 구조, 버전 이력
- [SCODA 아키텍처](scoda.md) — Scientific Collection Open Data Architecture, 패키지 규격, `kind` 필드
- [scoda-engine-core](scoda-engine-core.md) — stdlib-only 독립 PyPI Core 패키지 (S-2 분리)

### Features
- [시각화](visualization.md) — Tree Chart 엔진 (radial/rect/SBS/Diff/Morph/Timeline/Bar), Compound View
- [CRUD 프레임워크](crud-framework.md) — manifest-driven CRUD, Admin/Viewer, Preferences API
- [MCP 서버](mcp-server.md) — LLM 통합 (14 tools, stdio, Claude Desktop)
- [Meta-Package](meta-package.md) — paleobase 합성 트리, D3 radial tree
- [Multi-Package Serving](multi-package-serving.md) — URL prefix 라우팅, top-level 자동 리다이렉트

### Infrastructure
- [SCODA Hub](scoda-hub.md) — 정적 패키지 레지스트리 (GitHub Pages + Releases)
- [Docker 배포](docker-deployment.md) — 프로덕션 웹 뷰어 (gunicorn 단일 컨테이너)
- [릴리스 워크플로우](scoda-engine-release.md) — CI/CD, PyInstaller, MkDocs, bump_version

---

## PaperMeister

> Zotero + 로컬 폴더 + EndNote 등 이종 소스에서 논문을 수집·OCR·전문 검색하는 **로컬 연구 코퍼스 플랫폼**. 2026-03-30 ~ 2026-04-10 (12일) MVP 구축 + 장기 5-phase 로드맵 정립. [Noematica](noematica-brand.md) 생태계의 L1 제품.

### Overview & Architecture
- [PaperMeister 개요](papermeister-overview.md) — "Store first, understand later", 기술 스택, 12일 타임라인, 5-phase 로드맵, 6-layer 구현 스택
- [Sync-centric 아키텍처 & 데이터 모델](papermeister-architecture.md) — Source 카테고리, 3-layer canonical record, SCODA로 가는 6-layer 파이프라인, PaleoBase feedback loop

### Core Features
- [OCR 파이프라인 & 비용 최적화](papermeister-ocr-pipeline.md) — RunPod Chandra2, 병렬 OCR, batch retry/resume, pre-filter, 페이지 단위 OCR, reserved GPU 분석
- [LLM Bibliographic Extraction](papermeister-biblio-extraction.md) — Zotero 9,500편 ground truth, structured output schema, Claude/GPT 비교, 2-stage Haiku→Sonnet, PaperBiblio + vision pass
- [Zotero Integration](papermeister-zotero-integration.md) — Collection sync, pagination, standalone PDF, resync + hash matching, OCR JSON upload, standalone promote, write-back 정책
- [CLI & Desktop GUI](papermeister-cli-and-gui.md) — PyQt6 MVP, CLI 명령, P06 Desktop MVP feature 정의, P07 6-layer 구현 계획

### Product & Brand
- [제품 전략 (MVP, vs RAG, GTM)](papermeister-product-strategy.md) — Sellable MVP 정의, positioning, 5-phase roadmap, Phase 1 task breakdown, 세 고객층 (개인/연구실/기관)
- [**Noematica 브랜드 & Naming**](noematica-brand.md) — 회사 브랜드 결정, naming journey, 도메인 스택, 브랜드 아키텍처 (Noematica → PaperMeister/SCODA → PaleoBase/Trilobase)

---

## Meta
- [Source Processing Status](source-status.md) — raw/ 디렉토리별 인제스트 처리 현황

## Sources
- [raw/fsis2026-devlog/](../raw/fsis2026-devlog/) — 90 files (2026-02-24 ~ 2026-04-11)
- [raw/fsis2026-docs/](../raw/fsis2026-docs/) — 10 files (PROJECT_PROPOSAL, server_architecture, admin/user manuals, NK reports, import reports)
- [raw/trilobase-devlog/](../raw/trilobase-devlog/) — 103 files + ~150 archive (2026-02-04 ~ 2026-03-18)
- [raw/scoda-engine-devlog/](../raw/scoda-engine-devlog/) — 97 files (2026-02-19 ~ 2026-03-18)
- [raw/Modan2-devlog/](../raw/Modan2-devlog/) — 29 files (2025-08-28 ~ 2025-09-05)
- [raw/PaperMeister-devlog/](../raw/PaperMeister-devlog/) — 25 files (2026-03-30 ~ 2026-04-10)
- [raw/PaperMeister-docs/](../raw/PaperMeister-docs/) — 31 files (명세/기획/브랜드, + `naming/` 서브디렉토리)
