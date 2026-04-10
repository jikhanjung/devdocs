# 시각화 (트리, 차트, 비교, 애니메이션)

분류학 데이터를 다양한 방식으로 시각화하는 시스템.

## 뷰 타입 계보

```
Tree View (D3.js)
├── Radial Tree     ← P75, P76
├── Diff Tree       ← 114
├── Side-by-Side    ← 112, 113
└── Morphing        ← 115, P82

ICS Chart View      ← Phase 28-30
Timeline View       ← Phase 29

Compound View       ← 115, P82
├── Profile Comparison
│   ├── Diff Table
│   ├── Diff Tree
│   └── Animated Morphing
└── Statistics
```

## Radial Tree (방사형 트리)

동심원 링으로 분류 계층을 표시. D3.js cluster/tree 레이아웃 기반.

### 구현 변형
- **Canonical DB 용 (P76)**: parent_id 직접 사용, 단일 쿼리로 충분
- **Assertion DB 용 (P75)**: edge_cache 경유, 프로필별 별도 쿼리

### 기능
- depth_toggle: 표시 깊이 조절
- rank_radius: 랭크별 반경 설정
- context menu: subtree view, watch toggle
- orphan 필터링
- 텍스트 스케일링 컨트롤

## 프로필 비교 시각화 (R02 Phase 0-4)

두 classification profile 간 차이를 시각화하는 시스템.

### Phase 0+1: Diff Table (111)
- 프로필 간 moved/added/removed 분류군 비교
- 행 색상 코딩 (이동/추가/삭제)
- compare_view 매니페스트

### Phase 2: Side-by-Side Tree (112, 113)
- 두 프로필의 트리를 나란히 표시
- TreeChartInstance 클래스 추출로 다중 인스턴스 렌더링 (P81)
- 줌 동기화
- 성능 최적화

### Phase 3: Diff Tree (114)
- 통합 트리에서 moved/added/removed 노드 표시
- 이동 노드 re-parenting 로직
- ghost edge (원래 위치 점선)
- 색상 코딩된 링크
- 범례
- removed taxa 사이드 패널

### Phase 4: Animated Morphing (115, P82)
- 프로필 A → 프로필 B로의 애니메이션 전환
- loadMorph(), renderMorphFrame() 구현
- textScale 컨트롤
- radial/rectangular 통합
- 동적 genera_count 최적화 (1000x 성능 향상)

## Compound View 아키텍처

여러 sub-view를 단일 뷰로 결합:
- Profile Comparison compound view: table + tree + morphing
- from/to 프로필 선택 컨트롤
- sub-view별 파라미터 스코핑
- Statistics compound view

## ICS 지질연대 시각화

- ICS (International Commission on Stratigraphy) 데이터 임포트
- Period/Epoch/Age 셀렉터
- Genus 분포 타임라인
- 차트 뷰: 지질연대별 속 분포

## Tree Chart 리팩토링 (P81)

tree_chart.js의 클래스 기반 리팩토링:
1. TreeChartInstance 클래스 추출 (상태 캡슐화)
2. 이중 렌더링 지원
3. 인스턴스 관리
4. 줌 동기화

## Tree Search & Watch (P84)

- 검색 노드 수정
- context menu에서 watch 토글
- watch list 패널
- watched 노드 2× 확대
- diff view에서 removed taxa 패널

## 관련 페이지

- [scoda-engine](scoda-engine.md) — 뷰어 런타임
- [Assertion 모델](assertion-model.md) — classification profile
- [Trilobase 개요](trilobase-overview.md)

---
*Sources: 036-038, 102b, 103, 111-115, 131, P25-P26, P75, P76, P81, P82, P84, R02*
