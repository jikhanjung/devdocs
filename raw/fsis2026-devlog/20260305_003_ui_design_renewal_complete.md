# UI 디자인 리뉴얼 작업 완료

- **작업일**: 2026-03-05
- **계획 문서**: `devlog/20260305_P04_ui_design_renewal.md`
- **변경 규모**: 86개 파일, +1519 / -1180 lines

---

## 1. 테마 기반 구축 (Phase 1)

### 신규 파일 (3개)
| 파일 | 설명 |
|------|------|
| `static/geo-theme.css` | 공유 테마 CSS — CSS 변수(어스톤 팔레트), 컴포넌트 클래스 정의 |
| `fsis/static/fsis/fsis-theme.css` | FSIS 앰버/화석 액센트 (#b8860b 계열) |
| `kprdb/static/kprdb/kprdb-theme.css` | KPRDB 딥틸/해양 액센트 (#3d6b5e 계열) |

### 수정 파일 (6개)
- `fsis/templates/fsisbase.html` — Google Fonts(Noto Sans KR), 테마 CSS 링크, `<body class="fsis-theme">`
- `kprdb/templates/kprbase.html` — 동일 구조, `<body class="kprdb-theme">`
- `fsis/templates/fsisnavbar.html` — `bg-info` → `geo-navbar`, Bootstrap Icons 아이콘 추가, `<ll>` 오타 수정
- `kprdb/templates/kprnavbar.html` — `bg-primary` → `geo-navbar`, 아이콘 추가
- `fsis/templates/fsisfooter.html` — `geo-footer` 적용, 저작권 2021-2026
- `kprdb/templates/kprfooter.html` — 동일, 저작권 2022-2026

### CSS 컴포넌트 클래스 목록
- `.geo-navbar` — 그라데이션 배경, 둥근 모서리
- `.geo-page-header` — 섹션 헤더 (그라데이션 배경, 흰 텍스트)
- `.geo-card` — 콘텐츠 카드 (border, shadow, radius)
- `.geo-table` — 목록 테이블 (hover 효과, 정렬된 간격)
- `.geo-detail-table` — 상세/폼 테이블 (th 배경, 고정 폭)
- `.btn-geo-primary` / `.btn-geo-secondary` — 테마 버튼
- `.geo-search-bar` — 검색/필터 바 (독립 컴포넌트)
- `.geo-pagination` — 페이지네이션
- `.geo-footer` — 푸터
- `.geo-actions` — 버튼 그룹 (중앙 정렬, gap)
- `.geo-stat-card` — 대시보드 통계 카드
- `.geo-login-card` — 로그인 폼 카드 (중앙 정렬, max-width)

---

## 2. 헤더/테이블/버튼 일괄 교체 (Phase 2)

약 50개 콘텐츠 템플릿에 대해 기계적 치환 수행.

### 치환 패턴
| Before | After |
|--------|-------|
| `<h4 class="bd-title p-2 bg-secondary text-light">` | `<h4 class="geo-page-header"><i class="bi bi-icon"></i>` |
| `btn-info text-white` | `btn-geo-primary` |
| `btn-primary` (kprdb) | `btn-geo-primary` |
| `table table-striped table-sm` (목록) | `table table-sm geo-table` |
| `table table-striped table-sm` (상세/폼) | `table table-sm geo-detail-table` |

### 아이콘 매핑
- 표본 → `bi-gem`, `bi-list-ul`, `bi-pencil-square`
- 논문/참고문헌 → `bi-journal-text`
- 저자 → `bi-people`, 저널 → `bi-book`
- 학명 → `bi-tag`, 지층 → `bi-layers`, 지질시대 → `bi-bar-chart-steps`
- 변경내역 → `bi-clock-history`, 사용자 → `bi-person`
- 사진/그림 → `bi-image`, 다운로드 → `bi-download`

### base.html 의존 제거
- `fsis/templates/fsis/` 하위 22개 파일이 `base.html` → `fsisbase.html`로 전환됨
- `base.html`을 extends하는 템플릿이 더 이상 없음

---

## 3. 검색/페이지네이션 개선 (Phase 3)

### 검색 바 분리
- 기존: `<thead>` 안에 검색 폼 + 건수 정보 혼재
- 변경: `<div class="geo-search-bar">` 독립 컴포넌트로 분리
- 적용 대상: museum_specimen_list, reference_list, author_list, journal_list, scientificname_list, lithounit_list, museum_specimen_history_list 등

### 목록 페이지 카드화
- 전체 목록 페이지를 `.geo-card`로 감싸고, `<table>` 위에 `table-responsive` 래퍼 추가
- `<tfoot>` 대신 `.card-body` 안에 paginator + 액션 버튼 배치

### 페이지네이터 통일
- `fsis/templates/paginator.html` (구 버전) → `geo-pagination` + `btn-geo-primary` 스타일 적용
- `fsis/templates/fsispaginator.html` — 동일 적용
- `kprdb/templates/paginator.html` — 이미 적용 완료

---

## 4. 홈/특수 페이지 재설계 (Phase 4)

| 파일 | 변경 내용 |
|------|-----------|
| `fsis/templates/fsis/index.html` | `base.html` → `fsisbase.html`, 빈 페이지 → 4개 `.geo-stat-card` 대시보드 |
| `kprdb/templates/kprdb/intro.html` | SVG 아이콘 → Bootstrap Icons, `bg-primary` → `.geo-stat-card` |
| `fsis/templates/fsis/fossil_map.html` | 컨트롤 패널 어스톤 스타일링 (둥근 모서리, 그림자, accent color) |
| `fsis/templates/fsis/chrono_chart.html` | `.geo-card` + `table-responsive` 래퍼 |
| `kprdb/templates/kprdb/chronounit_chart.html` | 동일 |

---

## 5. 폼/상세 페이지 스타일링 (Phase 5)

약 35개 템플릿에 대해:
- `.geo-card` + `.card-body` 래퍼 추가
- 액션 버튼(편집/삭제/목록) → `.geo-actions`로 통일
- 로그인/비밀번호 변경 → `.geo-login-card` 중앙 정렬
- dashboard, history_overview → 각 섹션별 개별 `.geo-card`

---

## 6. 반응형/모바일 (Phase 6)

### geo-theme.css 미디어 쿼리
- `@media (max-width: 768px)` — navbar 전폭, 검색바 세로 정렬, 버튼 전폭, 페이지네이션 줄바꿈
- `@media (max-width: 576px)` — 상세 테이블 th/td 블록화, 목록 테이블 카드 레이아웃 (`data-label` 기반)

### data-label 적용
- `museum_specimen_list.html` — 7개 컬럼에 `data-label` 추가
- `reference_list.html` — 3개 컬럼에 `data-label` 추가

---

## 7. 정리 (Phase 7)

- `<ll>` 오타 → Phase 1 navbar 재작성 시 제거됨
- `fsis/templates/paginator.html` — 구 버전 → geo-pagination 스타일로 업그레이드
- `base.html` extends 의존 0개 (Phase 2에서 전환 완료)
- 저작권 연도 2021-2022 → 2021-2026 / 2022-2026

---

## 검증

- `python manage.py check` — 모든 Phase 완료 후 0 issues 확인
- 기존 JS 기능(Select2, Babylon.js, 카카오맵) 영향 없음 (CDN 로드 순서 유지)
