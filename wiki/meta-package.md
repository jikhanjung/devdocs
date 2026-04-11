# Meta-Package 지원

**Meta-package**는 `data.db`를 직접 포함하지 않고, 다른 SCODA 패키지를 논리적으로 조합하여 하나의 상위 분류 체계를 표현하는 패키지 종류다. 2026-03-18에 trilobase P89(설계)와 scoda-engine P31(런타임 지원)이 동시 완료되어, **paleobase** 메타 패키지(고생물학 전체 tree of life)가 첫 사례로 동작한다.

## 동기

trilobase/brachiobase/graptobase/chelicerobase/ostracobase 등 여러 분류군별 패키지가 존재할 때:
- 각 패키지는 자기 분류군의 완전한 트리 보유 (예: trilobase → Trilobita)
- 사용자는 "고생물학 전체 생명체 계통도" 하나의 뷰를 원함
- 해결책: Phylum/Kingdom 수준의 **뼈대 트리**(meta_tree)를 별도 파일로 두고, 각 서브트리를 원격/로컬 패키지에 바인딩

## `kind: "meta-package"` 필드

기존 SCODA 패키지 manifest에 `kind` 필드 추가:
- `"package"` (기본): 일반 `.scoda` (data.db 포함)
- `"meta-package"`: `data.db` 없음, `meta_tree.json` + `package_bindings.json` 포함

## 패키지 구조

```
paleobase-0.1.0.scoda
├── manifest.json              (kind: "meta-package", dependencies: [...])
├── metadata.json
├── meta_tree.json             # 상위 계층 트리 (Life → Eukaryota → Metazoa → Arthropoda ...)
├── package_bindings.json      # 메타노드 → 멤버 패키지 매핑
└── README.md
```

### `meta_tree.json` (예)

```json
{
  "nodes": [
    { "id": "life",       "label": "Life",        "rank": null,      "children": ["eukaryota"] },
    { "id": "eukaryota",  "label": "Eukaryota",   "rank": "Domain",  "children": ["metazoa"] },
    { "id": "metazoa",    "label": "Metazoa",     "rank": "Kingdom", "children": ["arthropoda", "brachiopoda", "hemichordata"] },
    { "id": "arthropoda", "label": "Arthropoda",  "rank": "Phylum",  "children": [] }
  ]
}
```

### `package_bindings.json` (예)

```json
{
  "arthropoda":  [
    { "package": "trilobita",   "root_taxon": "Arthropoda" },
    { "package": "chelicerata", "root_taxon": "Chelicerata" },
    { "package": "ostracoda",   "root_taxon": "Ostracoda" }
  ],
  "brachiopoda": [ { "package": "brachiopoda", "root_taxon": "Brachiopoda" } ],
  "hemichordata": [ { "package": "graptolithina", "root_taxon": "Graptolithina" } ]
}
```

## Core 구현 (Phase 1, scoda-engine P31)

`core/scoda_engine_core/scoda_package.py`:

- `kind` property: `manifest.get('kind', 'package')`
- `is_meta_package` property: `kind == 'meta-package'`
- `data.db` 추출 생략 (`db_path = None`)
- `verify_checksum()`: meta-package면 항상 True (data.db 없음)
- `record_count`: meta-package면 0
- `meta_tree` property: ZIP 내 `meta_tree.json` 파싱
- `package_bindings` property: ZIP 내 `package_bindings.json` 파싱

**테스트**: `TestMetaPackage` 클래스 9개 테스트 신규 추가 (232 tests).

## Registry 통합 (Phase 2)

`PackageRegistry`:
- `list_packages()`: meta-package 항목에 `kind` + `member_packages` (paleocore 제외한 의존 패키지 목록)
- `get_package()`: `kind`, `meta_tree`, `package_bindings`, `member_packages` 포함
- `get_db()`: meta-package에 대해 `:memory:` SQLite 생성 후 하위 패키지 multi-ATTACH
- `scan()`: 기존 로직 그대로 동작 (db_path=None이면 자연스럽게 등록)

### SQLite ATTACH 한도 고려

- 기본 한도 10개. paleobase 시나리오:
  - paleocore + trilobita + brachiopoda + graptolithina + chelicerata + ostracoda = 6 ATTACH
  - 여유 4개 (overlay 등)
- 현재는 한도 내. 미래에 패키지 증가 시 LRU detach 전략 고려.

## 합성 트리 API (Phase 3)

`scoda_engine/app.py`에 3개 엔드포인트 추가:

| 엔드포인트 | 용도 |
|---|---|
| `GET /api/{pkg}/meta/tree` | `meta_tree.json` 원본 반환 |
| `GET /api/{pkg}/meta/bindings` | `package_bindings.json` 원본 반환 |
| `GET /api/{pkg}/meta/composite-tree` | 합성 트리 (전체 / lazy expand) |

### `composite-tree` 동작

- `node_id` 파라미터 없음 → 전체 meta_tree 노드 구조 + 각 노드 바인딩 정보 + 가용 여부
- `node_id` 지정 → 해당 노드에 바인딩된 **각 패키지의 `classification_edge_cache`**에서 root_taxon children 조회, 패키지별로 그룹화하여 반환

### manifest 엔드포인트 분기 (Phase 4 버그 수정, 043)

- 문제: `response_model=ManifestResponse`가 meta-package 전용 필드(`kind`, `meta_tree`, `package_bindings`)를 필터링
- 해결: meta-package일 때 `JSONResponse`로 직접 반환하여 Pydantic 모델 필터링 우회

### 라우트 순서 버그 (043)

- 문제: `/meta/tree`, `/meta/bindings`, `/meta/composite-tree`가 catch-all `/{entity_name}/{entity_id}` **뒤**에 등록되어 매칭 안 됨
- 해결: meta 엔드포인트를 catch-all 앞으로 이동

## 프론트엔드 (Phase 4)

### Landing Page (`templates/landing.html`)

- Meta-package 카드: `bi-diagram-3` 아이콘 + **META 배지**
- `member_packages` 목록 표시 (예: "5 packages: trilobita, brachiopoda, ...")
- Meta-package를 카드 목록 상단에 정렬
- **Top-level 필터링** (043): dependency 전용 패키지는 landing에서 숨김 (→ [multi-package-serving.md](multi-package-serving.md))

### Package Page (`static/js/app.js`)

- `loadManifest()`에서 `kind: "meta-package"` 감지 → `isMetaPackage = true`
- `renderMetaPackageUI()`: `composite-tree` API 호출 → 트리 렌더링
- 초기 구현(042): 텍스트 목록 + 재귀 `buildMetaTreeNode()`
- 개선(043): **D3.js radial tree** (Darwin tree-of-life 스타일)

### D3 Radial Tree UI (03-18, 043)

- Meta 노드: 밝은 배경 rect (`#f0f0f0`) + 2줄 (rank 위, name 아래)
- 패키지 노드: 파란 원 (record count **log 스케일** 크기), 클릭으로 패키지 페이지 이동
- 비활성 Phylum (바인딩 없음): 작은 회색 원
- Taxon name uppercase first (`BRACHIOPODA` → `Brachiopoda`)
- 줌/팬 지원
- **Brownian motion 애니메이션**: maxDrift=12px, damping=0.99, jitter=0.02

### DOM 버그 수정 (043)

- `getElementById('main-content')` → 실제로는 class (`container-fluid main-content`) → `null` 반환
- `document.body` fallback → `innerHTML` 덮어쓰기로 전체 페이지 파괴
- 수정: 기존 `.view-container` 숨기고 새 meta 컨테이너를 sibling으로 추가

## 검증

- 232 tests passing (기존 223 + meta-package 9)
- trilobase `dist/` 디렉토리 통합 테스트:
  - `list_packages()`: paleobase가 `kind=meta-package`, `members=[trilobita, brachiopoda, graptolithina, chelicerata, ostracoda]`
  - `composite-tree node:arthropoda`: trilobita(Arthropoda→Trilobita), chelicerata(CHELICERATA→3 Classes), ostracoda(OSTRACODA→5 Orders)
  - `composite-tree node:brachiopoda`: BRACHIOPODA→5 children
  - `composite-tree node:hemichordata`: Graptolithina→5 children

## 관련 페이지

- [scoda-engine](scoda-engine.md) — 런타임 구현
- [scoda-engine-core](scoda-engine-core.md) — `ScodaPackage` + `PackageRegistry`
- [SCODA 아키텍처](scoda.md) — `kind` 필드
- [Multi-Package Serving](multi-package-serving.md) — top-level 자동 리다이렉트
- [시각화](visualization.md) — D3 radial tree 스타일
- [Trilobase Packages](packages.md) — paleobase, trilobita, brachiopoda, ostracoda, chelicerata, graptolithina

---
*Sources: P31, 042, 043 (2026-03-18). 설계 origin: trilobase P89.*
