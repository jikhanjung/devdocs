# 시각화 (Tree Chart 엔진 + 컴파운드 뷰)

scoda-engine의 Tree Chart 렌더링 엔진은 D3 계층 레이아웃과 Canvas+SVG 하이브리드로 구현된 고성능 시각화 엔진이다. 25일간(2026-02-28 ~ 2026-03-18) radial 설계부터 meta-package D3 radial 트리까지 단계적으로 발전했다.

## 뷰 계보

```
tree_chart.js (통합 엔진, TreeChartInstance 클래스)
├── Radial Layout           ← P20, 024 (2026-02-28 ~ 03-01)
├── Rectangular Cladogram   ← P23, 032 (03-02)
├── Side-by-Side            ← P24, 039 (03-07)
├── Diff Tree               ← 041 (03-07)
├── Morph Animation         ← P25, 042 (03-09)
└── Timeline Sub-view       ← P28, 030/033/034 (03-14)

compound view (tab 내부에 서브탭 + 로컬 컨트롤)
├── tree_chart_morph        display type
├── tree_chart_timeline     display type
└── bar_chart               display type (03-17)

meta-package 전용
└── D3 Radial Tree          Darwin tree-of-life 스타일 (03-18)
```

## Tree Chart 엔진 진화

### Phase 1: Radial Hierarchy Display (P20, 02-28 ~ 03-01)

- **설계**: D3.js lazy load, Canvas + SVG 하이브리드 렌더링, semantic LOD, 줌/검색/detail 연동
- **구현 강화** (024): pruned tree, collapse/expand, subtree view, 컨텍스트 메뉴
- 잘못된 `.scoda` 파일에 대한 `BadZipFile → ValueError` 변환 (025)

### Phase 2: 이중 레이아웃 (P23, 03-02)

- `radial.js` → `tree_chart.js`로 파일 리네이밍, 용어 정리
- **Radial + Rectangular 이중 레이아웃** 지원 (같은 엔진, 레이아웃 모드만 전환)
- 초기 rectangular cladogram은 D3 기본 tree 레이아웃 사용

### Phase 3: Bottom-Up 레이아웃 엔진 (033, 03-03)

- **문제**: d3.tree()는 leaf 간격이 데이터에 따라 가변적이어서 rectangular/radial 모두 겹침 발생
- **해결**: d3.tree() 제거 → **bottom-up 레이아웃 엔진** 자체 구현
  - Leaf를 균등 간격으로 먼저 배치
  - 상위 노드는 자식 평균 위치로 재귀 산출
  - 라벨 위치 분기 (leaf: 바깥쪽, internal: 위/왼쪽)
  - Rank별 X 정렬 (rectangular)
- Radial/rectangular 동일 원리 적용 → leaf 겹침 원천 방지
- LEAF_GAP 이후 축소 (12 → 6px, 036)

### Phase 4: TreeChartInstance 클래스 리팩토링 (P24, 03-07)

- **동기**: Side-by-Side 뷰를 위해 트리를 2개 동시 렌더링해야 함
- 전역 20+ 변수 → `TreeChartInstance` 클래스 인스턴스로 캡슐화
- 인스턴스 관리자(`instances` 맵) + 각 인스턴스 별 캔버스/SVG 오버레이
- Side-by-Side 5종 동기화: zoom, hover, depth, collapse, subtree
- Bitmap cache 성능 최적화 (화면 밖 리렌더 방지)

### Phase 5: Diff Tree 시각화 (041, 03-07)

두 profile 간 차이 표시:
- Status별 색상 (added/removed/moved/unchanged)
- Moved 노드 re-parenting: ghost edge (원래 위치 점선) + 새 위치 실선
- 범례 + tooltip
- Removed Taxa 사이드 패널 (클릭 시 zoom)

### Phase 6: Morph Animation (P25, 03-09)

- `tree_chart_morph` display 타입 추가
- `snapshotPositions()` → lerp 보간 → 재렌더링
- Look-ahead 캐싱, 각도 shortest-path 보간 (갑작스러운 점프 방지, 044)
- 렌더링 단순화: SVG label → canvas, bitmap scale
- `genera_count` 쿼리 최적화: 10,310ms → 9ms (1000x)
- Text scale A± / `[`, `]` 키 단축키 (0.3~5.0 범위, 029)

### Phase 7: Tree Search / Watch / Removed (P26, 03-11)

- **Search 수정**: zoom 좌표 보정, bounding box fit
- **Watch 기능**: 2× 확대 렌더, 금색 링, Watch 패널, SBS sync
- **Removed Taxa 패널**: diff/morph 모드, 클릭 시 zoom
- Morph collapse 버그 수정 (노드/엣지 안 숨겨지는 문제)
- Tree left-click expand 버그 수정 (`_children` 체크 순서)

### Phase 8: Visible Depth 슬라이더 (028, 03-13)

- Gear(⚙) 팝업 내 슬라이더로 전역 visible depth 제어
- Radial/rectangular/SBS/morph 전부 동기화
- Composite detail 쿼리 파라미터 누락 자동 채움 버그 수정 동반

### Phase 9: Timeline Sub-view (P28, 03-14)

- `tree_chart_timeline` display type 추가
- 다축 지원 (Geologic / Publication), 슬라이더 + `⏮◀⏸▶⏭⏩` 컨트롤 + look-ahead 캐싱
- Step 간 연쇄 morph 애니메이션
- 버그 수정: 축 전환 시 빈 트리 잔류 (`fullRoot = null`), play 빈 스텝 무한 루프 (`currentIdx` 갱신 보장)

### Phase 10: Timeline 모바일 + 녹화 (03-17)

- Timeline 컨트롤 2행 재편, 레이블 폭 고정, 모바일 하단 위치 조정
- **Timeline axis → compound sub-tab 승격**: Diff Table/Diff Tree와 동일 레벨 탭으로 표시
- **WebM 동영상 녹화**:
  - Offline (Chrome/Edge/Firefox): WebMWriter 프레임 단위 (1920×1080, 30fps, `timeline-animation.webm`)
  - Realtime (Safari): MediaRecorder 폴백
  - Morph 녹화와 동일 패턴 (`setupRecordCanvas` → 녹화 → `restoreCanvas`)
- 녹화 버튼 아이콘: `bi-circle-fill` 빨강, 녹화 중 흰↔빨강 깜박임

## Compound View 아키텍처 (P25, 03-09)

하나의 탭 안에 **로컬 컨트롤 + 서브탭**을 가진 복합 뷰:

```json
{
  "type": "compound",
  "sub_views": {
    "morph": { "display": "tree_chart_morph", ... },
    "timeline": { "display": "tree_chart_timeline", "axis_modes": [...] }
  },
  "default_sub_view": "morph"
}
```

- `loadCompoundView()` — compound 매니페스트 로드
- `switchCompoundSubView()` — 서브탭 전환, composite key 파싱(`${sk}__axis${i}`)
- Axis가 여러 개인 timeline은 axis마다 별도 서브탭으로 자동 전개
- 사용 예: Profile Comparison (Diff Table + Diff Tree + Morph), Statistics (Geologic Timeline + Publication Timeline + Diversity Chart)

## Bar Chart Sub-view (03-17)

새로 추가된 display 타입으로, 시대별 분류군 다양성을 stacked bar chart로 표시:

- D3 stacked bar chart (X: 카테고리, Y: 값, 색상: 그룹별)
- "Group by" 드롭다운으로 grouping rank 선택 (Order/Suborder/Superfamily/Family)
- Hover tooltip, HTML div 레전드 (스크롤), ResizeObserver 반응형
- `bar_chart_options`: `x_key`, `x_order_key`, `group_key`, `value_key`, `grouping_param`, `grouping_ranks`, `default_grouping`
- trilobase 측 쿼리: `diversity_by_age` (genus-only recursive CTE, `classification_edge_cache` profile 필터)

## Statistics 탭 재구성 (trilobase, 03-17)

기존 "Timeline" 단일 탭을 "Statistics" compound 탭으로 재편:
- **Geologic Timeline** — 지질시대별 트리 애니메이션 (단일 axis)
- **Publication Timeline** — 출판연도별 트리 애니메이션 (단일 axis)
- **Diversity Chart** — stacked bar chart

탭 순서: Tree → Comparison → Statistics

## 글로벌 Tree UI 요소

- **Tree 툴바 + 기어(⚙) 팝업**: Layout / Text size / Visible depth
- **Watch 패널**: 2× 확대된 노드 목록, 모바일에서 `left: 16px; top: 40px`
- **Removed Taxa 패널**: diff/morph/timeline 모드에서 접기/펼치기 토글
- **Tree 드로어 (모바일)**: overlay + 슬라이드인 + backdrop + 항목 선택 시 자동 닫힘
- **Search 입력창**: 모바일에서 `max-width: 30vw`로 자동 축소

## Meta-Package D3 Radial Tree (03-18)

paleobase meta-package 전용 뷰. tree_chart.js와 독립적으로 구현:

- **Darwin tree-of-life 스타일** D3 radial tree
- Meta 노드: 밝은 배경 rect (`#f0f0f0`) + 2줄 (rank 위, name 아래)
- 패키지 노드: 파란 원 (record count log 스케일 크기), 클릭으로 패키지 이동
- 비활성 Phylum(바인딩 없음): 작은 회색 원
- Taxon name uppercase first (`BRACHIOPODA` → `Brachiopoda`)
- 줌/팬, Brownian motion 애니메이션 (maxDrift=12px, damping=0.99, jitter=0.02)

→ 상세: [Meta-Package](meta-package.md)

## 오프라인 지원 (031, 03-14)

`static/vendor/` 번들로 D3, Bootstrap, Bootstrap Icons 로컬 내장 (~755KB). CDN 없이 기관 내부망에서도 동작.

## 관련 페이지

- [scoda-engine](scoda-engine.md) — 뷰어 런타임
- [SCODA 아키텍처](scoda.md) — manifest, display type
- [Compound / Multi-Package Serving](multi-package-serving.md)
- [Meta-Package](meta-package.md) — D3 radial tree
- [Assertion 모델](assertion-model.md) — classification profile (trilobase)
- [Trilobase 개요](trilobase-overview.md)

---
*Sources: raw/scoda-engine-devlog/ — P20, 024-025, 032-033, 036, P23, P24, 039-041, P25, 042, P26, 042-044, 028-029, P28, 030-036, P30, 036-037, 039-041, P31, 042-043. 추가 컨텍스트: raw/trilobase-devlog/ (P75, P76, P81, P82, P84, R02, 111-115).*
