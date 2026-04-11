# SCODA (Scientific Collection Open Data Architecture)

과학 데이터 컬렉션을 자기 기술적(self-describing), 불변(immutable), 선언적(declarative) 패키지로 배포하는 아키텍처.

## SCODA 6계층

| 계층 | 테이블/구성요소 | 역할 |
|---|---|---|
| **Core** | artifact_metadata, provenance, schema_descriptions | 패키지 정체성, 출처, 스키마 설명 |
| **Display Intent** | ui_display_intent, ui_queries | 6가지 표시 의도, 14+ 저장 쿼리 |
| **Saved Queries** | ui_queries | 이름 기반 SQL 쿼리 (파라미터 지원) |
| **UI Manifest** | ui_manifest | 선언적 뷰 정의 (JSON) |
| **Release** | release.py | SHA-256 해시, 불변 버전 패키지 |
| **Local Overlay** | user_annotations (별도 DB) | 정전 데이터와 분리된 사용자 주석 |

## .scoda 패키지 형식

```
{package_id}-{version}.scoda (ZIP)
├── {name}-{version}.db          # SQLite 정전 DB (meta-package는 없음)
├── manifest.json                # 뷰/쿼리 정의 + kind + dependencies
├── metadata.json                # 패키지 메타데이터
├── README.md                    # 설명
├── CHANGELOG.md                 # 변경 이력 (Keep a Changelog 형식)
├── checksums.sha256             # 무결성 검증
├── meta_tree.json               # meta-package 전용 (선택)
└── package_bindings.json        # meta-package 전용 (선택)
```

### `kind` 필드 (2026-03-18 도입)

- `"package"` (기본값): 일반 SCODA 패키지, `data.db` 포함
- `"meta-package"`: `data.db` 없이 다른 패키지들을 합성하는 상위 레벨 패키지 (예: paleobase)
  - `meta_tree.json`, `package_bindings.json` 포함
  - `verify_checksum()` skip, `record_count = 0`
  - 하위 패키지 multi-ATTACH로 조회
  - 상세: [Meta-Package](meta-package.md)

### Hub Manifest
- `{package_id}-{version}.manifest.json`: SCODA Hub Registry 등록용
- SHA-256 해시, 의존성 선언, 패키지 메타데이터
- GitHub Release에 자동 업로드
- 파일명에 버전 포함 규칙 (2026-02-25, 017)
- 상세: [SCODA Hub](scoda-hub.md)

## 매니페스트 시스템

### Display Intents (6가지)
tree, table, detail, chart, timeline, radial

### View 정의 (JSON)
```json
{
  "view_id": "taxonomy_tree",
  "display_type": "tree",
  "query": "taxonomy_tree_nodes",
  "columns": [...],
  "label_map": { "opinion_type별 동적 라벨" }
}
```

### Global Controls
- 프로필 셀렉터: 분류 프로필 런타임 전환
- localStorage 지속성
- 파라미터 주입을 통한 쿼리 파라미터화

### 선언적 UI 매니페스트
- 완전 선언적 뷰 정의 (tree, table, detail, chart, timeline)
- linked_table 섹션: FK 기반 연결 테이블 표시
- sub_queries: 상세 페이지 내 하위 쿼리
- compound views: 여러 sub-view 결합
- editable_entities: CRUD 가능 엔티티 선언

### label_map
opinion_type별 동적 컬럼 라벨:
- PLACED_IN → "Proposed Parent"
- SPELLING_OF → "Correct Spelling"
- SYNONYM_OF → "Senior Taxon"

## 매니페스트 검증

- JSON 스키마 기반 validator (manifest_validator.py)
- 스키마 정규화 (manifest_schema_normalization)
- 자동 검색(auto-discovery) + fallback 메커니즘

## Spec-Implementation 동기화 (S-5, 2026-02-22)

scoda-engine 독립 후 SCODA 스펙과 런타임 코드 사이 5개 갭 해소:

1. **Checksum-on-load 검증** — `ScodaPackage` 로드 시 자동 SHA-256 검증
2. **Dependency `required` 필드** — optional/required 구분
3. **SemVer range 파싱** — `">=X,<Y"` AND 형식, `>=`, `<`, `,` 연산자
4. **예외 클래스 3개** — `ScodaDependencyError`, `ScodaChecksumError`, `ScodaManifestError`
5. **CHANGELOG.md 지원** — Keep a Changelog 형식, 패키지에 포함

추가로 Boolean 라벨 통일(`true_label`/`false_label`, 전역 기본값), `label_map` 동적 컬럼 라벨(S-6) 반영.

`validate_manifest.py`는 S-3(2026-02-22)에서 `scoda-engine-core`로 이동하여 trilobase 측 중복 제거. 상세: [scoda-engine-core](scoda-engine-core.md).

## 오버레이 시스템

- 정전 DB: 읽기 전용 (불변)
- 오버레이 DB: 읽기/쓰기 (별도 파일)
- user_annotations: notes, corrections, alternatives, links
- entity_name 추적: 크로스 버전 마이그레이션

### PyInstaller 오버레이 문제
- 임시 폴더 내 오버레이 DB 내구성 문제
- 해결: 오버레이 DB를 EXE 외부에 자동 생성

## 버전 관리

- Semantic Versioning: Major(스키마 변경) / Minor(데이터 추가) / Patch(품질 수정)
- bump_version.py: artifact_metadata + create_scoda.py 의존성 자동 업데이트
- CHANGELOG.md: Keep a Changelog 형식
- versioned DB filename: `{name}-{version}.db`, db_path.py semver 자동 검색

## 관련 페이지

- [scoda-engine](scoda-engine.md) — SCODA 뷰어/서버 런타임
- [scoda-engine-core](scoda-engine-core.md) — 독립 stdlib-only Core 패키지
- [SCODA Hub](scoda-hub.md) — 정적 레지스트리
- [Meta-Package](meta-package.md) — kind=meta-package
- [Trilobase 개요](trilobase-overview.md)
- [Packages](packages.md) — SCODA 기반 패키지들
- [PaperMeister 아키텍처](papermeister-architecture.md) — PaperMeister corpus → SCODA 6-layer 파이프라인의 upstream
- [Noematica 브랜드](noematica-brand.md) — SCODA는 Noematica 생태계의 L2 제품

---
*Sources (trilobase archive): 012-018, 020-021, 030, 049, 052, 074-076, 085, 089, 094-095, 104, P07-P12, P20, P58-P59, P65, P67, P77. Sources (scoda-engine): P05, P06, P08, P09, 005-007, 017, P31, 042-043.*
