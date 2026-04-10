# UI 디자인 리뉴얼 계획 — 자연/지질 테마

**날짜**: 2026-03-05

## Context

현재 사이트는 Bootstrap 5 기본 스타일(회색 bg-secondary 헤더, 단순 table-striped)로 되어 있어 시각적 정체성이 없고 올드한 느낌. 지질유산 DB답게 자연/지질 테마(어스톤 컬러, 화석/암석 분위기)로 전체 90개 템플릿을 리뉴얼.

- fsis(화석표본)와 kprdb(고생물문헌)는 별도 디자인 유지하되 공통 테마 공유
- Bootstrap 5 유지, 커스텀 CSS 테마 레이어 추가

---

## Phase 1: 테마 기반 구축 (CSS + Base 템플릿)

모든 페이지에 자동 적용되는 기반 작업. 이 단계만으로도 전체 분위기가 바뀜.

### 새 파일 생성
| 파일 | 설명 |
|------|------|
| `static/geo-theme.css` | 공유 테마 (CSS 변수, 컴포넌트 클래스) |
| `fsis/static/fsis/fsis-theme.css` | FSIS 앰버/화석 액센트 |
| `kprdb/static/kprdb/kprdb-theme.css` | KPRDB 딥틸/해양 액센트 |

### 핵심 디자인 요소
- **색상 체계**: CSS 변수로 어스톤 팔레트 정의 (stone, amber, forest, earth)
- **FSIS 액센트**: 앰버/골드(#b8860b) — 화석 느낌
- **KPRDB 액센트**: 딥틸(#3d6b5e) — 해양/퇴적 느낌
- **폰트**: Noto Sans KR (Google Fonts CDN)
- **컴포넌트 클래스**: `.geo-page-header`, `.geo-card`, `.geo-table`, `.geo-detail-table`, `.btn-geo-primary`, `.geo-search-bar`, `.geo-pagination`, `.geo-navbar`, `.geo-footer`

### 수정 파일 (6개)
- `fsis/templates/fsisbase.html` — 테마 CSS 링크 추가, 폰트 로드
- `kprdb/templates/kprbase.html` — 동일
- `fsis/templates/fsisnavbar.html` — `bg-info` → `geo-navbar`, 아이콘 추가
- `kprdb/templates/kprnavbar.html` — `bg-primary` → `geo-navbar`, 아이콘 추가
- `fsis/templates/fsisfooter.html` — `geo-footer` 스타일
- `kprdb/templates/kprfooter.html` — 동일

---

## Phase 2: 헤더/테이블/버튼 일괄 교체 (~50개 파일)

기계적 치환 작업. 모든 콘텐츠 페이지에 테마 적용.

### 치환 패턴
```html
<!-- 헤더: bg-secondary → geo-page-header -->
<h4 class="bd-title p-2 bg-secondary text-light"> → <h4 class="geo-page-header">

<!-- 테이블: geo-card로 감싸기 -->
<div class="geo-card">
  <h4 class="geo-page-header">제목</h4>
  <table class="table table-sm geo-table">...</table>
</div>

<!-- 버튼 -->
btn-info text-white → btn-geo-primary (FSIS)
btn-primary → btn-geo-primary (KPRDB)
```

---

## Phase 3: 네비게이션/검색/페이지네이션 개선 (~20개 파일)

- 검색 폼을 `<thead>`에서 분리 → `.geo-search-bar`로 독립
- 페이지네이터 스타일 개선 (`fsispaginator.html`, `kprdb/paginator.html`)
- navbar 로그인 폼 스타일링

---

## Phase 4: 홈/특수 페이지 재설계 (~6개 파일)

- `fsis/templates/fsis/index.html` — 카드 기반 대시보드 (현재 거의 빈 페이지)
- `kprdb/templates/kprdb/intro.html` — 기존 카드 레이아웃에 테마 색상 적용
- `fsis/templates/fsis/fossil_map.html` — 컨트롤 패널만 어스톤 스타일링
- `base.html` 의존 제거 (index.html, chrono_chart.html → fsisbase.html로 전환)

---

## Phase 5: 폼/상세 페이지 정리 (~30개 파일)

- 폼 페이지: `geo-card` + `geo-detail-table` 적용
- 상세 페이지: 시각적 계층 정리, 사진/3D 영역 카드화
- 로그인/사용자 페이지: 중앙 정렬 카드 레이아웃

---

## Phase 6: 반응형/모바일 (~10개 파일)

- `mobile.css` 재작성 (현재 `data-label` 속성 없어서 미작동)
- Bootstrap `table-responsive` 래퍼 추가
- 모바일 미디어 쿼리 추가 (카드/헤더/네비바 간격 조정)

---

## Phase 7: 정리

- `style.css` 사이드바 스타일 제거
- `fsisnavbar.html` `<ll>` 오타 수정
- 미사용 `paginator.html` 정리
- 저작권 연도 갱신

---

## 검증 방법

- 각 Phase 완료 후 `python manage.py runserver`로 주요 페이지 확인:
  - 목록: `/fsis/museum_specimen_list/`, `/kprdb/reference_list/`
  - 상세: 표본 상세, 문헌 상세
  - 폼: 표본 추가/수정
  - 특수: `/fsis/fossil_map/`, 연대표
- Select2, Babylon.js 3D 뷰어, 카카오맵 등 기존 JS 기능 정상 동작 확인
- 모바일 뷰포트(Chrome DevTools) 확인
