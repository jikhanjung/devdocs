# Source Processing Status

`raw/` 하위 각 디렉토리의 인제스트 처리 현황.

| 디렉토리 | 파일 수 | 소스 날짜 범위 | 마지막 처리 시각 (KST) | 처리 범위 | 상태 |
|---|---|---|---|---|---|
| `raw/fsis2026-devlog/` | 90 | 2026-02-24 ~ 2026-04-11 | 2026-04-11 | 전체 (90/90) | **완료** |
| `raw/fsis2026-docs/` | 10 | 제안서·서버 아키텍처·매뉴얼·NK 리포트 | 2026-04-11 | 전체 (10/10) | **완료** |
| `raw/trilobase-devlog/` | 103 + 150 (archive) | 2026-02-04 ~ 2026-03-18 | 2026-04-10 23:30 | 전체 (253/253) | **완료** |
| `raw/scoda-engine-devlog/` | 97 | 2026-02-19 ~ 2026-03-18 | 2026-04-11 | 전체 (97/97) | **완료** |
| `raw/Modan2-devlog/` | 29 | 2025-08-28 ~ 2025-09-05 | 2026-04-11 | 전체 (29/29) | **완료** |
| `raw/PaperMeister-devlog/` | 25 | 2026-03-30 ~ 2026-04-10 | 2026-04-11 | 전체 (25/25) | **완료** |
| `raw/PaperMeister-docs/` | 31 | — (명세/기획/브랜드) | 2026-04-11 | 전체 (31/31) | **완료** |

> **다음 인제스트 시**: 마지막 처리 시각 이후에 추가/수정된 파일만 처리하면 됩니다.
> `find raw/{dir} -newer wiki/source-status.md -name '*.md'` 로 대상 파일을 확인할 수 있습니다.

## 처리 이력

### fsis2026-devlog
- **2026-04-10 23:05 KST**: 초기 배치 인제스트. 89개 파일 전량 처리. 11개 wiki 페이지 생성 (overview, kprdb, ghdb, evaluator, pdf-pipeline, task-management, map-system, deployment, data-import, ui-design, django-migration).
- **2026-04-11 KST (오후)**: Delta 인제스트 (1 devlog + 10 docs). 068 devlog (저널 dedup + 학명 이탤릭 + 저널 논문 수). **주요 발견**: 시스템 공식명 **KOFHIN** (Korea Fossil Heritage Inventory), 프로덕션 URL `https://fsis.psok.or.kr`. 2개 신규 페이지 생성 (kofhin-manuals, north-korea-data). 6개 기존 페이지 업데이트 (overview는 PROJECT_PROPOSAL 사업 맥락·Brilha 정확 가중치·P0~P4 매트릭스·ID 체계·해외 벤치마크 반영; deployment는 server_architecture의 GCP·Nginx·host cron·umask·파일 우선순위·3 env 파일 반영; evaluator는 Brilha 정확 가중치와 Sevieri 2020 P0~P4 매트릭스·좌표 공개 수준 반영; data-import는 Import 보고서 집계·5단계 PDF 전략·068 저널 dedup 반영; kprdb는 068 이탤릭·저널 count 반영; pdf-pipeline은 `.pdf.extract.claude.json` 우선순위 명시). index.md 섹션 타이틀을 "FSIS2026 / KOFHIN"으로 갱신.

### trilobase-devlog
- **2026-04-10 23:30 KST**: 초기 배치 인제스트. 253개 파일 전량 처리 (main 103 + archive 150). 9개 wiki 페이지 생성 (trilobase-overview, scoda, scoda-engine, assertion-model, taxonomy-data, visualization, mcp-server, packages, treatise-extraction).

### scoda-engine-devlog
- **2026-04-11 KST**: 초기 배치 인제스트. 97개 파일 전량 처리. DEVLOG_SUMMARY.md 전체 읽기 + 10+ 대표 raw 파일 샘플링. 7개 신규 페이지 생성 (scoda-engine-core, scoda-hub, crud-framework, docker-deployment, multi-package-serving, meta-package, scoda-engine-release). 4개 기존 페이지 대규모 업데이트 (scoda-engine, visualization, scoda, mcp-server). index.md에 새로운 "SCODA Engine" top-level 섹션 추가 — scoda/scoda-engine/visualization/mcp-server를 Trilobase 하위에서 SCODA Engine으로 이동. 25일 스프린트(02-19 ~ 03-18)의 S-1 ~ S-7 마일스톤, Hub 시스템, Radial→SBS→Diff→Morph→Timeline→BarChart Tree Chart 진화, CRUD 프레임워크, 프로덕션 Docker, Multi-Package Serving v0.3.0, Mobile UI, Meta-Package(paleobase)까지 모두 커버.

### Modan2-devlog
- **2026-04-11 KST**: 초기 배치 인제스트. 29개 파일 전량 처리. DEVLOG_SUMMARY.md 없음 — 10개 대표 raw 파일 샘플링 (001/007/010/011/012/015/022/023/027/028/030 + 014/024). 6개 신규 페이지 생성 (modan2-overview, modan2-architecture, modan2-testing, modan2-analysis, modan2-infrastructure, modan2-stability). index.md에 새로운 "Modan2" top-level 섹션 추가 (Trilobase와 SCODA Engine 사이). 9일 코드 품질 현대화 스프린트(2025-08-28 ~ 2025-09-05): Modan2.py 1500줄 → 900줄 MVC 리팩토링(011), 0 → 192 pytest 자동화(028), QSettings → JSON 설정 마이그레이션(012), print → logging(014), 전역 try/except 에러 핸들링(015), version.py Single Source of Truth + bump_version.py(020-022), PCA+CVA+MANOVA 통합 실행 + 3D 차원 버그 수정(023), OpenGL/GLUT + xvfb CI 호환성(030), WSL 썸네일 동기화 known issue(029).

### PaperMeister-devlog / -docs
- **2026-04-11 KST (오후)**: 초기 배치 인제스트. devlog 25 + docs 31 = 56개 파일 전량 처리. `docs/index.md`를 TOC로 활용 + 13개 대표 파일 샘플링 (P01/010/P06/P07, specification/sync_centric/data_model/sellable_mvp/papermeister_vs_rag/final_name_direction/noematica_brand_package 등). 8개 신규 페이지 생성 (papermeister-overview, papermeister-architecture, papermeister-ocr-pipeline, papermeister-biblio-extraction, papermeister-zotero-integration, papermeister-cli-and-gui, papermeister-product-strategy, noematica-brand). index.md에 새로운 "PaperMeister" top-level 섹션 추가 (SCODA Engine 뒤). **핵심 발견**: 회사 브랜드 **Noematica** 결정이 docs/naming/에 있음 — PaperMeister + SCODA + PaleoBase + Trilobase를 포괄하는 umbrella. noematica-brand.md는 현재 PaperMeister 섹션 하위이지만 미래 lint 때 top-level 이동 후보. PaperMeister 아키텍처는 "canonical corpus layer above source systems" 원칙 + SCODA로 가는 6-layer 파이프라인 + PaleoBase feedback loop.
