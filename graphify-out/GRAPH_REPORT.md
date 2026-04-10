# Graph Report - raw/trilobase-devlog  (2026-04-10)

## Corpus Check
- Large corpus: 251 files · ~192,058 words. Semantic extraction will be expensive (many Claude tokens). Consider running on a subfolder, or use --no-semantic to run AST-only.

## Summary
- 617 nodes · 740 edges · 42 communities detected
- Extraction: 86% EXTRACTED · 14% INFERRED · 0% AMBIGUOUS · INFERRED: 107 edges (avg confidence: 0.77)
- Token cost: 0 input · 0 output

## God Nodes (most connected - your core abstractions)
1. `Trilobase Database` - 18 edges
2. `Classification Profile` - 16 edges
3. `SCODA Package Format` - 13 edges
4. `Paleocore Database` - 11 edges
5. `Assertion-Centric Taxonomy Model` - 10 edges
6. `MCP Server (Model Context Protocol)` - 10 edges
7. `R01: Managing Time-Varying Taxonomy` - 8 edges
8. `P89: Paleobase Meta-Package Design` - 8 edges
9. `ui_queries Named Query Table` - 8 edges
10. `Assertion DB UI Parity Plan (P74b)` - 7 edges

## Surprising Connections (you probably didn't know these)
- `P53: Assertion-Centric Canonical Taxonomic Data Model` --semantically_similar_to--> `user_annotations Table (note/correction/alternative/link)`  [INFERRED] [semantically similar]
  raw/trilobase-devlog/archive/20260216_P53_assertion_centric_model.md → raw/trilobase-devlog/archive/20260207_P09_local_overlay.md
- `buildHierarchy() Unified Function` --semantically_similar_to--> `Generic SCODA Package Viewer Plan (P36)`  [INFERRED] [semantically similar]
  raw/trilobase-devlog/archive/20260216_P55_hierarchy_view_unification.md → raw/trilobase-devlog/archive/20260213_P36_generic_scoda_viewer.md
- `Overlay Database (User Annotations)` --conceptually_related_to--> `Trilobase Database`  [INFERRED]
  raw/trilobase-devlog/archive/20260209_P14_phase22_mcp_wrapper.md → raw/trilobase-devlog/20260219_079_db_dist_reorganization.md
- `Radial Tree Visualization Display` --semantically_similar_to--> `Side-by-Side Tree Chart`  [INFERRED] [semantically similar]
  raw/trilobase-devlog/20260228_P75_radial_tree_visualization_plan.md → raw/trilobase-devlog/20260307_112_side_by_side_tree.md
- `Side-by-Side Tree Chart` --semantically_similar_to--> `Animated Tree Morphing`  [INFERRED] [semantically similar]
  raw/trilobase-devlog/20260307_112_side_by_side_tree.md → raw/trilobase-devlog/20260309_P82_animated_morphing_plan.md

## Hyperedges (group relationships)
- **Trilobase Data Quality Fix Group (Feb 2026)** — 20260222_P66_group_a_fix_agnostida, 20260222_P66_spelling_of_opinion, 20260219_081_fix_unlinked_synonyms, 20260228_098_fix_country_id [INFERRED 0.82]
- **Paleobase Unified Package Ecosystem** — 20260315_P88_paleobase_unified, 20260316_130_tsf_mass_extraction, 20260318_135_paleobase_bindings_fix, 20260318_136_version_bump [INFERRED 0.78]
- **Tree Visualization Feature Development** — 20260307_112_side_by_side_tree, 20260307_113_sbs_sync_perf, 20260309_P82_animated_morphing, 20260228_P75_radial_tree [INFERRED 0.85]
- **Treatise Classification Import Pipeline (TXT→JSON→AssertionDB)** — 116_treatise1959_txt_pipeline, P78_treatise_import_plan, 120_assertion_db_020_source_build, 132_treatise2004_to_1997_rename [EXTRACTED 0.92]
- **Assertion DB UI and Data Parity Convergence Sprint** — 101_P74b_assertion_ui_parity, 115_compound_view_morphing, 083_taxon_bibliography_junction, 094_hub_manifest_generation [INFERRED 0.78]
- **Paleobase Multi-Package Integration (Meta Tree + Bindings + PaleoCore)** — P89_paleobase_meta_package_design, 139_meta_tree_structure_fixes, concept_paleocore, concept_meta_tree [EXTRACTED 0.90]
- **Repository Split and CI/CD Infrastructure Setup** — 20260219_078_repo_split, 20260219_P62_repo_split_plan, 20260223_P68_ci_cd, 20260219_078_scoda_engine_repo [INFERRED 0.82]
- **Classification Profile Diff Visualization Suite** — 20260302_R02_tree_diff_viz, 20260307_111_profile_diff_table, 20260307_P81_side_by_side, 20260307_111_compare_mode [EXTRACTED 0.90]
- **Assertion-Centric DB Build Pipeline Evolution** — 20260228_P74_assertion_centric, 20260311_P83_build_assertion, 20260311_P83_source_txt_pipeline, 20260228_P74_assertion_schema [INFERRED 0.85]
- **Treatise Sources to Classification Profile Build Pipeline** — ref_treatise1959, ref_treatise2004_ch4, ref_treatise2004_ch5, R03_comprehensive_removal_logic, trilobase_assertion_db [INFERRED 0.85]
- **Paleobase Meta-Package Assembly (Stage 0-1)** — 134_paleobase_stage0_stage1, 134_paleobase_bindings, paleobase_meta_tree_json, 137_metazoa_root [EXTRACTED 0.90]
- **Taxonomy Profile Diff Visualization Suite** — 114_diff_tree, 114_profile_diff_edges_query, P84_removed_taxa_panel, 114_diff_status_color_coding [INFERRED 0.80]
- **Radial Tree 구현 파이프라인 (P75→P76→P87)** — 102b_P75_radial_tree_assertion, 103_P76_radial_tree_canonical, 123_P87_timeline, 102b_scoda_engine_radial [INFERRED 0.85]
- **Trilobase 0.3.0 통합 작업 집합** — 121_trilobase_030_consolidation, 122_handoff_030_update, 122_assertion_centric_schema, 121_classification_edge_cache [EXTRACTED 0.90]
- **Treatise 프로필 체인 (default→1959→2004)** — 110_treatise1959_standalone, 105_treatise2004_profile, 105_hybrid_edge_cache, 121_classification_edge_cache [EXTRACTED 0.90]
- **SCODA Desktop Application Evolution: Flask → FastAPI + MCP Merge** — concept_flask_app, concept_fastapi_app, concept_mcp_server, concept_scoda_desktop [INFERRED 0.85]
- **Phase 46 Runtime Purification: Generic Endpoint + Dynamic MCP Tools + Legacy Removal** — 20260214_P40_phase46_runtime_purification, concept_composite_endpoint, concept_mcp_tools_json, concept_scoda_runtime_generic [EXTRACTED 0.95]
- **Phase C UID Population Using External APIs** — 20260215_P44_uid_population_phase_c, concept_crossref_api, concept_macrostrat_api, concept_uid_schema [EXTRACTED 0.90]
- **SCODA Core Components (Identity + Provenance + Release + Immutability)** — P07_artifact_metadata_table, P07_provenance_table, P08_release_py, P07_scoda_version_rules [EXTRACTED 0.95]
- **MCP Server Deployment Architecture (SSE + stdio + Two-EXE)** — 023_mcp_sse_server, P19_two_exe_split, 030_asynccontextmanager_pattern [EXTRACTED 0.90]
- **Manifest-Driven UI Stack (ui_manifest + generic renderer + auto-generate fallback)** — P07_ui_manifest_table, P33_open_detail_renderer, 069_auto_generate_manifest [EXTRACTED 0.92]
- **PaleoCore SCODA Packaging Pipeline** — 043_phase35_paleocore_scoda, 045_phase37_build_paleocore_scoda, p30_phase36_combined_scoda_test [EXTRACTED 0.95]
- **ICS Chronostratigraphic Integration (Import + UI + Mapping)** — p24_ics_chronostrat_import, p24_temporal_ics_mapping_table, p25_ics_web_ui [EXTRACTED 0.95]
- **Manifest Schema Evolution (tree/chart → hierarchy)** — 014_ui_manifest_table, 076_hierarchy_view_type, p59_normalize_view_def [EXTRACTED 0.90]
- **SCODA Desktop Runtime Components** — app_py, gui_py, mcp_server_py, scoda_package_py, 20260214_P39_scoda_desktop_package [EXTRACTED 0.95]
- **Trilobase/PaleoCore Database Separation** — trilobase_db, 20260213_P27_paleocore_db, 20260213_041_phase33_pc_prefix, 20260213_P27_paleocore_schema [EXTRACTED 0.90]
- **GUI and MCP EXE Deployment Variants** — 20260210_028_trilobase_exe, 20260210_028_trilobase_mcp_exe, 20260210_P16_mcp_stdio_single_exe, 20260210_P17_gui_mcp_sse_optional [INFERRED 0.80]
- **FastAPI Migration: Flask→FastAPI + Pydantic Models + Structured Logging** — 20260215_P46_fastapi_migration, 20260215_067_pydantic_response_model, 20260217_P57_structured_logging [INFERRED 0.85]
- **UID Population Phase A: Plan + Countries Cleanup + Implementation** — 20260215_P42_uid_population_phase_a_plan, 20260212_031_countries_data_quality, 20260215_060_uid_population_phase_a [INFERRED 0.90]
- **Generic SCODA Viewer: Package Format + Registry + Reference SPA + Declarative UI** — 20260212_P20_scoda_package_format, 20260213_052_phase42_generic_scoda_viewer, 20260214_053_phase44_reference_spa, 20260213_049_phase39_declarative_manifest_ui [EXTRACTED 0.90]
- **SCODA Manifest-Driven Architecture Stack** — 051_ui_manifest_table, 013_ui_queries_table, 055_composite_api, 057_scoda_desktop_generic_viewer [INFERRED 0.88]
- **SCODA Package & DB Layer** — 030_scoda_format, 030_scoda_package_class, 020_canonical_db, 030_overlay_db [INFERRED 0.85]
- **Trilobase Taxonomy Schema Evolution** — 007_taxonomic_ranks_original, p03_taxonomic_ranks_table, p60_taxonomic_opinions_table, p60_is_placeholder_column [INFERRED 0.82]
- **SCODA Three-Database Architecture (Canonical + Overlay + PaleoCore)** — p12_canonical_db, p12_overlay_db, phase31_paleocore_db_file [EXTRACTED 0.95]
- **Trilobase Data Pipeline (Validation → Normalization → Relation Tables)** — phase3_data_validation, phase5_db_normalization, p04_formation_location_relations [INFERRED 0.85]
- **Manifest-Driven UI System (tree_options + chart_options + validator)** — p35_tree_options, p35_chart_options, p58_validate_manifest_py [INFERRED 0.80]

## Communities

### Community 0 - "Taxonomy Schema & Hierarchy"
Cohesion: 0.05
Nodes (52): UI Fixes: Statistics Dedup & Children Navigation, adrain2011.txt (Taxonomic Source Data), Phase 7: Order Integration & Taxonomic Hierarchy, taxonomic_ranks Table (Original Self-Referential), artifact_metadata Table, provenance Table, schema_descriptions Table, Phase 13: SCODA-Core Metadata Tables (+44 more)

### Community 1 - "Data Quality & Rebuild Pipeline"
Cohesion: 0.07
Nodes (51): 082: Data Quality Fixes — Colon, BRAÑA Encoding, Hyphen Linebreaks, 083: Taxon Bibliography Junction Table, 099: Modular Rebuild Pipeline Complete (35/35), 100: Rebuild DB vs Reference DB Diff Resolution, 101: P74b Assertion DB UI Feature Parity, 115: Profile Comparison Compound View and Morphing Animation, 116: Treatise 1959 TXT Pipeline and Bug Fixes, 120: Assertion DB 0.2.0 Source-Driven Build (+43 more)

### Community 2 - "Treatise Source Import Pipeline"
Cohesion: 0.07
Nodes (46): OCR Genus Extraction from Treatise 1959 PDF, 109: Treatise 1959 Taxonomy Extraction and Import, 119: Canonical Source Data File Generation, JA2002 Parser (5115 genera), Phase 16: Release Mechanism, DB Parsing Error Fixes and Genus Additions, Countries Table Data Quality Cleanup, Web UI Detail Pages and Cross-Links (+38 more)

### Community 3 - "Geographic & Formation Data"
Cohesion: 0.06
Nodes (42): Fix PaleoCore Chronostratigraphy Chart Sort Order, Phase 10: Formation/Location Relation Tables Plan, genus_formations Junction Table, genus_locations Junction Table, chart_options Manifest Schema Extension, Plan Phase 41: Manifest-Driven Tree & Chart Rendering, tree_options Manifest Schema Extension, Flask to FastAPI Migration Plan (+34 more)

### Community 4 - "Build & Deployment"
Cohesion: 0.05
Nodes (40): scripts/build.py (Build Automation), Phase 18: Standalone Executable Complete (PyInstaller), trilobase.spec (PyInstaller Configuration), country_cow_mapping Table (142 Records), cow_states Table (244 Records), scripts/import_cow.py, Phase 26: COW State System Membership v2024 Import, Phase 34: DROP PaleoCore Tables from trilobase.db (+32 more)

### Community 5 - "Web Interface & MCP Server"
Cohesion: 0.11
Nodes (29): Phase 11: Web Interface Complete, Phase 11: Web Interface Plan, GUI Control Panel (Phase 19), Phase 22 MCP Server Implementation, Phase 22: MCP Server Plan, Phase 25: MCP stdio Single EXE, GUI MCP SSE UI Removal, SCODA ZIP Package Format Plan (P20) (+21 more)

### Community 6 - "GUI & Desktop App"
Cohesion: 0.09
Nodes (29): GUI Log Viewer Complete (Phase 21), Flask Server as Subprocess Pattern, GUI Log Viewer Plan (Phase 21), trilobase.exe (GUI Executable), trilobase_mcp.exe (MCP stdio Executable), Two EXE Split (GUI + MCP stdio), MCP SSE Integration Plan (Phase 23), MCP stdio Single EXE Plan (P16) (+21 more)

### Community 7 - "Treatise Taxonomy Extraction"
Cohesion: 0.07
Nodes (28): Treatise 2004 Chapter 4: Order Agnostida JSON, Treatise 2004 Chapter 5: Order Redlichiida JSON, 102: Treatise 2004 Chapter 4 and 5 Taxonomy Extraction, import_treatise1959.py Script, Diff Status Color Coding (same/moved/added/removed), 114: Diff Tree Single-Merge Tree Visualization, profile_diff_edges SQL Query, convert_to_source_format.py Script (+20 more)

### Community 8 - "Assertion DB & Radial Tree"
Cohesion: 0.08
Nodes (28): Agnostida Order (id=5341), Group A 철자 변형 중복 해소 + Agnostida Order 생성, P75: Assertion DB Radial Tree 구현, radial_tree_edges 쿼리 (classification_edge_cache), scoda-engine Radial Display 고도화, P76: Canonical SCODA Radial Tree 구현, radial_tree_nodes 쿼리 (canonical DB), taxonomic_ranks 테이블 (parent_id 내장) (+20 more)

### Community 9 - "SCODA Package & UI Manifest"
Cohesion: 0.11
Nodes (27): 092: rank_detail Children Table Bug Fix and Redirect, 094: Hub Manifest Auto-Generation, ICS Chronostratigraphic Chart View Plan (P26), Phase 38: SCODA Desktop Rebranding, Phase 39 Declarative Manifest UI Migration, Phase 40: CORS + SPA Example + EXE Rename, Phase 42 Generic SCODA Package Viewer, P37: Control Panel Single Package Serving (+19 more)

### Community 10 - "Core Database Schema"
Cohesion: 0.09
Nodes (25): Phase 4: Database Creation and Schema, synonyms Table (899 synonym records), taxa Table (5,113 trilobite genera), temporal_ranges Table (geological time codes), families Table (181 trilobite families), Phase 6: Family Normalization, Proposal: Bibliography-Taxa Link (genus_bibliography table), artifact_metadata Table (identity, version, license) (+17 more)

### Community 11 - "PaleoCore DB & Queries"
Cohesion: 0.08
Nodes (24): scripts/add_scoda_ui_tables.py, genus_locations Table Data Fix, PaleoCore DB (paleocore.db), 093: UI Queries pc.* Prefix Fix and genus_locations Data Cleanup, Formation/Location Parsing Improvement (suffix-based), P72: Source Text → DB Full Rebuild Pipeline Design, scripts/rebuild_database.py (Modular Pipeline), P74: Assertion-Centric Test DB Plan and Implementation (+16 more)

### Community 12 - "SCODA Engine Repo Split"
Cohesion: 0.11
Nodes (20): 078: SCODA Engine / Trilobase Repository Split, scoda-engine Repository, P62: SCODA Engine / Trilobase Repo Split Plan, scoda-engine pip Package (pyproject.toml), P68: CI/CD GitHub Actions Plan, GitHub Actions CI Workflow (.github/workflows/ci.yml), GitHub Actions Release Workflow (.github/workflows/release.yml), P85: Base Taxonomy Template for SCODA Packages Plan (+12 more)

### Community 13 - "Formation-Location Relations"
Cohesion: 0.11
Nodes (18): Phase 10: Formation/Location Relation Tables Complete, genus_formations Table (Genus-Formation Many-to-Many), genus_locations Table (Genus-Country Many-to-Many), Phase 12: Data Cleanup and UI Improvements, GET /api/rank/<id> Rank Detail API, _auto_generate_manifest() (schema-based manifest fallback), GET /api/{entity_name}/{entity_id} Catch-All Endpoint, 069: Generic Viewer Domain-Agnostic Refactor (+10 more)

### Community 14 - "Agnostida Classification"
Cohesion: 0.12
Nodes (16): Agnostida-Trilobita Placement Opinion (JA2002 vs A2011), 087: SPELLING_OF Opinion Type and Agnostida Restructure, Temporal Code Auto-Fill (85 valid genera), Agnostina Suborder Addition (id=5344), 088: Valid Genus parent_id NULL Resolution, taxonomic_opinions questionable assertion_status, 125: Brachiobase v0.2.2 Temporal Code and Timeline, Temporal Code Extraction from PDF (PyMuPDF) (+8 more)

### Community 15 - "Early Data Cleaning"
Cohesion: 0.14
Nodes (16): Phase 1: Line Normalization, Genus List Structural Changes Log, Phase 12: Bibliography Table Complete, Phase 12: References Table Plan, Phase 27: Geographic Regions Hierarchy, P44: UID Population Phase C, P50: Taxonomic Opinions DB Design, bibliography Table (+8 more)

### Community 16 - "Dynamic MCP & Paleobase"
Cohesion: 0.14
Nodes (14): Phase 46 Step 2: Dynamic MCP Tool Loading, mcp_tools.json (패키지별 동적 MCP 도구 정의), ScodaPackage / PackageRegistry 클래스, Paleobase Package Registry, SCODA Package: bryozoa, SCODA Package: mollusca, TSF 기반 신규 패키지 6개 생성, Taxonomic Source Format (TSF) (+6 more)

### Community 17 - "Taxonomy Consolidation"
Cohesion: 0.18
Nodes (14): Phase 2: Character Encoding Fixes, Phase 8: Taxonomy Table Consolidation, Phase 9: Taxa and Taxonomic Ranks Consolidation, taxonomic_ranks Unified Table, paleocore.db (Shared Infrastructure DB), PaleoCore Schema Design Plan (P27), Formation Metadata Backfill (Phase 64), Formations Table (paleocore.db) (+6 more)

### Community 18 - "MCP SSE Integration"
Cohesion: 0.18
Nodes (12): GUI Integration: Flask + MCP Dual Start, MCP Server SSE Mode (Starlette + Uvicorn, port 8081), Phase 23: MCP Server SSE Integration, asynccontextmanager Session Pattern (pytest anyio fix), 030: MCP Test asyncio Compatibility Fix, 073: Structured Logging Introduction, TkLogHandler: Python logging.Handler to GUI Log Panel, P10: Standalone Executable (PyInstaller) (+4 more)

### Community 19 - "Overlay DB Separation"
Cohesion: 0.24
Nodes (11): Canonical DB (trilobase.db, read-only), scripts/init_overlay_db.py, trilobase_overlay.db, Plan: Separate Overlay DB (Option 1), Phase 17: Local Overlay (User Annotations), user_annotations Table, entity_name as Matching Anchor Pattern, id_mapping.json Release Artifact (+3 more)

### Community 20 - "UI Manifest Schema"
Cohesion: 0.29
Nodes (8): scripts/add_scoda_manifest.py, Phase 15: UI Manifest (Declarative View Definition), ui_manifest Table, hierarchy View Type (Replaces tree/chart), 076: Manifest Schema Normalization — DB Level (A-3), scripts/validate_manifest.py, P59: Manifest Schema Normalization — DB Level (A-3) Plan, normalizeViewDef() Backward Compatibility Function

### Community 21 - "Taxonomy Table Evolution"
Cohesion: 0.32
Nodes (8): families Table (Dropped After Consolidation), taxonomic_ranks Table (Consolidated Hierarchy), Phase 8: Taxonomy Table Consolidation, is_placeholder Column on taxonomic_ranks, Trigger: trg_sync_parent (parent_id Auto-Sync), Taxonomic Opinions Design Option A' (Additive Table + Integrity Safeguards), P52: Taxonomic Opinions — Final Design Document, taxonomic_opinions Table

### Community 22 - "Taxonomic Opinions"
Cohesion: 0.29
Nodes (7): label_map 동적 컬럼 레이블 기능, opinion_type (PLACED_IN / SYNONYM_OF / SPELLING_OF), Synonyms → Taxonomic Opinions 마이그레이션, taxonomic_opinions 테이블 (통합 의견 저장소), Trilobase 데이터 정제 및 DB 구축 계획, Jell & Adrain 2002 PDF 소스, 초기 taxa/synonyms 스키마 설계

### Community 23 - "Paleobase Meta-Package"
Cohesion: 0.4
Nodes (6): paleobase_bindings.json (Package Bindings), Paleobase Meta-Package (.scoda), 134: Paleobase Stage 0-1 Implementation, Metazoa as Meta Tree Root, 137: Paleobase Meta Tree Refinement, paleobase_meta_tree.json

### Community 24 - "Tree Diff Visualization"
Cohesion: 0.5
Nodes (5): R02: Tree Diff Visualization Roadmap, Compare Mode UI (compareMode Global State), 111: Profile Diff Table — Compare Mode Phase 0+1, P81: Side-by-Side Tree Chart Refactoring Plan, TreeChartInstance Class Refactoring

### Community 25 - "Community 25"
Cohesion: 0.4
Nodes (5): Plan P18: Remove MCP SSE UI from GUI, scripts/gui.py (MCP SSE UI Removal), create_mcp_app() Factory Function, FastAPI app.mount('/mcp', starlette_app), Plan P48: MCP + Web API Single Process Integration

### Community 26 - "Community 26"
Cohesion: 0.4
Nodes (5): Phase: Flask App Tests (test_app.py), data/ Directory (Source Data Files), Phase 45: Directory Restructure (Runtime/Data Separation), scoda_desktop/ Package, tests/ Directory (Separated Test Suite)

### Community 27 - "Community 27"
Cohesion: 0.5
Nodes (4): Fide Author Matching Improvement, genus_detail Manifest sub_queries Fix, 097: Synonym Manifest and Fide Fix (Post-Migration), taxon_bibliography Table (fide entries)

### Community 28 - "Community 28"
Cohesion: 0.67
Nodes (3): 080: Fix NULL Parent 13 Genera, 084: NULL Parent Families to Order Uncertain, Order Uncertain (id=144) Placeholder Node

### Community 29 - "Community 29"
Cohesion: 0.67
Nodes (3): bump_version.py Script, 085: Version Management and Changelog Process, P65: Version Changelog Process Plan

### Community 30 - "Community 30"
Cohesion: 0.67
Nodes (3): P77: DB 파일명 버전 포함 통일, scripts/bump_version.py, scripts/db_path.py (버전 탐색 모듈)

### Community 31 - "Community 31"
Cohesion: 0.67
Nodes (3): Phase 29: ICS Chronostratigraphy Web UI, Phase 30: ICS Chart View, ics_chronostrat Table

### Community 32 - "Community 32"
Cohesion: 1.0
Nodes (2): MkDocs Documentation Site Plan (P69), MkDocs Documentation Site

### Community 33 - "Community 33"
Cohesion: 1.0
Nodes (2): 133: Package Rename to Taxonomic Names, Package Rename: trilobase → trilobita

### Community 34 - "Community 34"
Cohesion: 1.0
Nodes (2): 090: CI/CD GitHub Actions Implementation, GitHub Actions Release Workflow (v*.*.* tag trigger)

### Community 35 - "Community 35"
Cohesion: 1.0
Nodes (2): PaleoCore 테이블 분리 (Phase 34), taxa_count 컬럼 참조 제거 버그픽스

### Community 36 - "Community 36"
Cohesion: 1.0
Nodes (2): Phase 13: UI Filter and Navigation Improvements, Valid-Only Genus Filter (showOnlyValid toggle)

### Community 37 - "Community 37"
Cohesion: 1.0
Nodes (1): Watch Node Feature (2x Rendering)

### Community 38 - "Community 38"
Cohesion: 1.0
Nodes (1): Future Work Proposals (Phase 1-12 Complete)

### Community 39 - "Community 39"
Cohesion: 1.0
Nodes (1): Proposal: Temporal Code Normalization

### Community 40 - "Community 40"
Cohesion: 1.0
Nodes (1): Proposal: Statistics Dashboard (Chart.js / D3.js)

### Community 41 - "Community 41"
Cohesion: 1.0
Nodes (1): Fix Sticky Table Header (UI Bug Fix)

## Knowledge Gaps
- **193 isolated node(s):** `Manual Release Manifest Fix (#095)`, `MkDocs Documentation Site Plan (P69)`, `Agnostida Order`, `Profile Comparison Tab (Compound View)`, `SPELLING_OF Opinion Type` (+188 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `Community 32`** (2 nodes): `MkDocs Documentation Site Plan (P69)`, `MkDocs Documentation Site`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 33`** (2 nodes): `133: Package Rename to Taxonomic Names`, `Package Rename: trilobase → trilobita`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 34`** (2 nodes): `090: CI/CD GitHub Actions Implementation`, `GitHub Actions Release Workflow (v*.*.* tag trigger)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 35`** (2 nodes): `PaleoCore 테이블 분리 (Phase 34)`, `taxa_count 컬럼 참조 제거 버그픽스`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 36`** (2 nodes): `Phase 13: UI Filter and Navigation Improvements`, `Valid-Only Genus Filter (showOnlyValid toggle)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 37`** (1 nodes): `Watch Node Feature (2x Rendering)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 38`** (1 nodes): `Future Work Proposals (Phase 1-12 Complete)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 39`** (1 nodes): `Proposal: Temporal Code Normalization`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 40`** (1 nodes): `Proposal: Statistics Dashboard (Chart.js / D3.js)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 41`** (1 nodes): `Fix Sticky Table Header (UI Bug Fix)`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Trilobase Database` connect `Treatise Source Import Pipeline` to `Data Quality & Rebuild Pipeline`, `Web Interface & MCP Server`?**
  _High betweenness centrality (0.032) - this node is a cross-community bridge._
- **Are the 2 inferred relationships involving `Trilobase Database` (e.g. with `Overlay Database (User Annotations)` and `SCODA Stable UID Schema v0.2`) actually correct?**
  _`Trilobase Database` has 2 INFERRED edges - model-reasoned connections that need verification._
- **Are the 2 inferred relationships involving `SCODA Package Format` (e.g. with `092: rank_detail Children Table Bug Fix and Redirect` and `UI Manifest (ui_manifest)`) actually correct?**
  _`SCODA Package Format` has 2 INFERRED edges - model-reasoned connections that need verification._
- **Are the 3 inferred relationships involving `Assertion-Centric Taxonomy Model` (e.g. with `099: Modular Rebuild Pipeline Complete (35/35)` and `083: Taxon Bibliography Junction Table`) actually correct?**
  _`Assertion-Centric Taxonomy Model` has 3 INFERRED edges - model-reasoned connections that need verification._
- **What connects `Manual Release Manifest Fix (#095)`, `MkDocs Documentation Site Plan (P69)`, `Agnostida Order` to the rest of the system?**
  _193 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Taxonomy Schema & Hierarchy` be split into smaller, more focused modules?**
  _Cohesion score 0.05 - nodes in this community are weakly interconnected._
- **Should `Data Quality & Rebuild Pipeline` be split into smaller, more focused modules?**
  _Cohesion score 0.07 - nodes in this community are weakly interconnected._