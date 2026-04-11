# Wiki Log

## [2026-04-11] lint | overview.md 분리 — repo-wide vs FSIS2026
- **Action**: 기존 `wiki/overview.md`가 레포 전체가 아닌 FSIS2026/KOFHIN 한 프로젝트만 다루고 있어서 이름과 역할이 어긋나 있었음. 파일명을 `fsis2026-overview.md`로 rename하고 (`modan2-overview.md`, `papermeister-overview.md`, `trilobase-overview.md` 네이밍 패턴과 일치), 새 `wiki/overview.md`를 레포 전체의 meta-overview로 작성.
- **Renamed**: `overview.md` → `fsis2026-overview.md` (git mv로 history 보존)
- **Pages created**: 1 (`overview.md` — repo meta-overview: 5개 프로젝트 표, Noematica 생태계 vs 독립 프로젝트 관계, 레포 구조, 워크플로우)
- **Pages updated**: 12 (index.md에 Repo Overview 섹션 추가 및 FSIS2026 링크 텍스트 명확화; fsis2026-overview.md 상단에 meta-overview 포인터 추가; 그리고 overview.md를 참조하던 10개 파일 — kofhin-manuals, django-migration, data-import, deployment, ui-design, evaluator, ghdb, north-korea-data, map-system, task-management — 에서 링크를 fsis2026-overview.md로 업데이트)
- **Index updated**: Yes (맨 위에 Repo Overview 섹션 신설)
- **Notes**: 기존 overview.md 제목 "FSIS2026 / KOFHIN 프로젝트 개요"가 이미 범위를 드러내고 있었고, 다른 프로젝트들(Trilobase/Modan2/PaperMeister)은 `*-overview.md` 규약을 따르는데 FSIS2026만 generic한 `overview.md`를 점유하고 있어서 네이밍이 비일관적이었음. 사용자가 이 불일치를 지적해서 수정. 새 repo overview는 Noematica 생태계(PaperMeister → SCODA Engine → Trilobase/PaleoBase + 도메인 feedback loop)와 독립 프로젝트(FSIS2026 용역, Modan2 도구) 두 계열의 관계를 명시적으로 그려준다.

## [2026-04-11] ingest | fsis2026 delta (1 devlog + 10 docs)
- **Source**: raw/fsis2026-devlog/ (+1 file, 068 journal cleanup, 2026-04-11) + raw/fsis2026-docs/ (10 files, NEW)
- **Action**: Incremental delta ingest — previous fsis2026-devlog already processed; 10 docs files never seen before (full ingest); 1 devlog added
- **Pages created**: 2 (kofhin-manuals, north-korea-data)
- **Pages updated**: 6 (overview, deployment, evaluator, data-import, kprdb, pdf-pipeline)
- **Index updated**: Yes (section title changed to "FSIS2026 / KOFHIN", surfaced official production name, added links to 2 new pages + data-import Import 보고서 집계 + sources list updated to include fsis2026-docs)
- **Notes**: 🔑 **Key discovery**: system is officially named **KOFHIN** (Korea Fossil Heritage Inventory), production URL https://fsis.psok.or.kr. Previously this wiki used only the internal "FSIS2026" codename. PROJECT_PROPOSAL.md provided the authoritative sources for: (1) 국가유산청 contract context (8-month, 4 goals, GeoSite-ID/Specimen-ID/Reference-ID axes, 5-phase execution), (2) exact Brilha 2016 weights (35/20/20/15/10) and P0~P4 Risk×Value×Feasibility matrix, (3) international benchmarks (iDigBio/Smithsonian/ROM/NHM). server_architecture.md provided specifics for deployment.md (GCP IP 34.64.158.160, Nginx 443/8001 routing, host-side cron in /srv/fsis2026/venv, Claude extract file priority `.pdf.extract.claude.json` → `.pdf.extract.json`, IMAGE_TAG 0.3.16, 3 env files). NK author mapping and PDF lists are substantial enough for a dedicated page (north-korea-data.md) that cross-links back to data-import.md as a KPRDB subcorpus. Admin + user manuals (680+ lines combined) consolidated into kofhin-manuals.md. Devlog 068 (journal cleanup + scientific name italic + ref count) slotted into data-import.md (journal dedup) and kprdb.md (italic/count).

## [2026-04-11] ingest | PaperMeister-devlog + PaperMeister-docs (batch)
- **Source**: raw/PaperMeister-devlog/ (25 files, 2026-03-30 ~ 2026-04-10) + raw/PaperMeister-docs/ (31 files + naming/ subdirectory)
- **Action**: Initial batch ingest of PaperMeister — two sources combined (devlog + docs)
- **Pages created**: 8 (papermeister-overview, papermeister-architecture, papermeister-ocr-pipeline, papermeister-biblio-extraction, papermeister-zotero-integration, papermeister-cli-and-gui, papermeister-product-strategy, noematica-brand)
- **Pages updated**: 0 (fully new project)
- **Index updated**: Yes (new "PaperMeister" top-level section added after SCODA Engine, with Overview/Architecture, Core Features, Product/Brand subsections)
- **Notes**: docs/ contained a TOC (`docs/index.md`) covering 8 categories — used as the map. Sampled ~13 files (5 docs + 8 devlog/docs) strategically including the pivot point (010 Project Overview), P07 (most recent 6-layer plan), P04 (LLM biblio strategy), and the naming session (final_name_direction_memo + noematica_brand_package). Key architectural insight: PaperMeister is positioned as a "canonical corpus layer" above external source systems (Zotero/EndNote/dirs), with a clear 6-layer pipeline to SCODA via domain extraction. Business/brand dimension is substantial — docs include pricing, customer discovery, GTM, vs-RAG positioning, and a complete company brand decision (**Noematica**) that spans PaperMeister + SCODA + PaleoBase + Trilobase. `noematica-brand.md` is placed under PaperMeister for now but may be promoted to a top-level "Brand / Meta" section in a future lint pass. Devlog filename numbering is scrambled (planning docs P01-P07 and implementation logs 001-018 share a sequence but are interleaved by date).

## [2026-04-11] ingest | Modan2-devlog (batch)
- **Source**: raw/Modan2-devlog/ (29 files, 2025-08-28 ~ 2025-09-05)
- **Action**: Initial batch ingest of the Modan2 9-day code modernization sprint
- **Pages created**: 6 (modan2-overview, modan2-architecture, modan2-testing, modan2-analysis, modan2-infrastructure, modan2-stability)
- **Pages updated**: 0 (fully new project — no existing Modan2 references in wiki)
- **Index updated**: Yes (new "Modan2" top-level section added between Trilobase and SCODA Engine)
- **Notes**: No DEVLOG_SUMMARY.md present. Read ~10 strategic samples (001 proposal, 007 initial test completion, 010 46KB master plan, 011 MVC completion, 012 QSettings→JSON, 015 error handling, 022 version mgmt completion, 023 analysis debugging, 027 test restructuring, 028 warning resolution, 030 final compat/CI fixes, plus 014/024 for context). Sprint converted a 1,500-line monolith to MVC (40% reduction), built testing from 0 → 192 tests with xvfb CI, modernized config/logging/versioning/build, and fixed 8 analysis bugs in a single session (023). Open questions: no raw file for "016" (sequence gap, possibly unused); Phase 5-6 of master plan (custom widgets, plugin system) remain unimplemented; 37 skipped tests still outstanding at sprint end.

## [2026-04-11] ingest | scoda-engine-devlog (batch)
- **Source**: raw/scoda-engine-devlog/ (97 files, 2026-02-19 ~ 2026-03-18)
- **Action**: Initial batch ingest of the standalone scoda-engine repo devlog (post trilobase-split)
- **Pages created**: 7 (scoda-engine-core, scoda-hub, crud-framework, docker-deployment, multi-package-serving, meta-package, scoda-engine-release)
- **Pages updated**: 4 (scoda-engine, visualization, scoda, mcp-server — major rewrites to cover v0.1.x → v0.3.4+ scope)
- **Index updated**: Yes (new "SCODA Engine" top-level section with Architecture / Features / Infrastructure subsections)
- **Notes**: 25-day sprint spanning S-1 (generic fixture) → S-7 (CI/CD) → Hub → Radial → CRUD → Prod Docker → Tree Chart refactor (SBS/Diff/Morph/Timeline) → Multi-Package Serving v0.3.0 → Mobile UI → Bar Chart → Meta-Package (paleobase). Read DEVLOG_SUMMARY.md + sampled 10+ raw files (P01, 004, 013, 017, 029, 039-041, 042/03-14, 042/03-18, 043, P31). scoda-engine and related pages moved from under "Trilobase" into new "SCODA Engine" top-level section.

## [2026-04-10] ingest | trilobase-devlog (batch)
- **Source**: raw/trilobase-devlog/ (103 files + ~150 archive files, 2026-02-04 ~ 2026-03-18)
- **Action**: Batch ingest of all Trilobase development logs (main + archive)
- **Pages created**: 8 (trilobase-overview, scoda, scoda-engine, assertion-model, taxonomy-data, visualization, mcp-server, packages, treatise-extraction)
- **Pages updated**: index.md (restructured with FSIS2026/Trilobase sections)
- **Index updated**: Yes
- **Notes**: ~253 source files spanning 46+ development phases. Organized into 1 overview, 3 architecture pages, 2 data pages, and 3 feature pages with full cross-linking.

## [2026-04-10] ingest | fsis2026-devlog (batch)
- **Source**: raw/fsis2026-devlog/ (89 files, 2026-02-24 ~ 2026-04-10)
- **Action**: Initial batch ingest of all FSIS2026 development logs
- **Pages created**: 11 (overview, kprdb, ghdb, evaluator, pdf-pipeline, task-management, map-system, deployment, data-import, ui-design, django-migration)
- **Index updated**: Yes
- **Notes**: First wiki population. Source files span 67 implementation logs, 19 planning documents, 1 review document. Organized into 4 entity pages and 7 concept pages with full cross-linking.
