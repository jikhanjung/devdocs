# Wiki Index

---

## FSIS2026

### Project Overview
- [FSIS2026 프로젝트 개요](overview.md) — 전체 프로젝트 구조, 기술 스택, 타임라인, 데이터 규모

### Entity Pages (앱/시스템)
- [KPRDB](kprdb.md) — 한국 고생물학 논문 데이터베이스 앱 (Reference, Author, Journal 모델)
- [GHDB](ghdb.md) — 2026 지질유산 데이터베이스 앱 (화석/암석 표본 관리)
- [Evaluator](evaluator.md) — 화석산지 위험도 평가 시스템 (Brilha 5항목 모델)

### Concept Pages (주요 시스템)
- [PDF 처리 파이프라인](pdf-pipeline.md) — OCR → 정규식 추출 → Claude AI 보강 3단계 처리
- [작업 관리 시스템](task-management.md) — 교수-학생 논문 처리 작업 할당/추적, 3 Phase 워크플로우
- [지도 시스템](map-system.md) — Kakao Maps → Leaflet → MapLibre GL JS 전환 이력
- [배포 인프라](deployment.md) — Docker, Nginx, SSL, 백업, cron, 환경변수 관리
- [데이터 수집](data-import.md) — jpaleodb, DBpia, CrossRef, 북한논문 등 외부 소스 임포트
- [UI 디자인 시스템](ui-design.md) — Earth-tone 테마, Bootstrap 5, 컴포넌트, 네비게이션
- [Django 마이그레이션](django-migration.md) — nkfadmin(Django 3.1) → fsis2026(Django 5.2) 전환 과정

---

## Trilobase

### Project Overview
- [Trilobase 프로젝트 개요](trilobase-overview.md) — 삼엽충 분류학 DB, 기술 스택, 타임라인, 버전 이력

### Architecture
- [SCODA 아키텍처](scoda.md) — Scientific Collection Open Data Architecture, 패키지 규격, 매니페스트
- [scoda-engine](scoda-engine.md) — 범용 SCODA 뷰어/서버 런타임, FastAPI, PyInstaller
- [Assertion 모델](assertion-model.md) — 다중 분류 의견 관리, classification profiles, edge cache

### Data
- [분류학 데이터](taxonomy-data.md) — Phase 1-12 데이터 정제, rebuild 파이프라인, 품질 수정
- [Treatise 추출](treatise-extraction.md) — Treatise 1959/1997 OCR 추출, R04 TXT 형식, TSF 대규모 추출

### Features
- [시각화](visualization.md) — Radial tree, diff tree, side-by-side, morphing, compound view
- [MCP 서버](mcp-server.md) — LLM 통합 (14 tools, stdio, Claude Desktop)
- [Multi-package](packages.md) — brachiobase, graptobase, paleobase, paleocore

---

## Meta
- [Source Processing Status](source-status.md) — raw/ 디렉토리별 인제스트 처리 현황

## Sources
- [raw/fsis2026-devlog/](../raw/fsis2026-devlog/) — 89 files (2026-02-24 ~ 2026-04-10)
- [raw/trilobase-devlog/](../raw/trilobase-devlog/) — 103 files + ~150 archive (2026-02-04 ~ 2026-03-18)
- [raw/scoda-engine-devlog/](../raw/scoda-engine-devlog/) — 97 files (2026-02-19 ~ 2026-03-18) *(미처리)*
