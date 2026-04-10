# 067 — 논문 상세검색, 테이블 레이아웃 개선, 페이지네이션 추가

**날짜**: 2026-04-10

## 작업 내용

### 1. 논문 목록 상세검색 (`reference_list`)
- 기존 출처(source) 드롭다운 제거
- 기본 검색어(제목+저자) `contains` → `icontains` (대소문자 무시)
- "상세검색" 토글 버튼 추가 (Bootstrap collapse)
- 상세검색 필드:
  - 연도 (from ~ to) — 드롭다운, DB 최소 연도~올해 초기값
  - 저자, 제목, 저널 (텍스트, icontains)
  - 문헌유형 (드롭다운: JN/BK/BS/RP/TH)
  - PDF 유무 (드롭다운: 전체/있음/없음)
  - 화석종류 (체크박스: 거화석/미화석/생흔화석)
- 모든 필터 GET 파라미터, 정렬/페이지네이션과 AND 조합
- `filter_qs` 문자열로 정렬/페이지 링크에서 검색 조건 유지
- paginator를 인라인으로 교체 (기존 공용 paginator.html은 다른 페이지에서 사용 중)
- 상세검색 파라미터 있으면 자동 펼침 + 초기화 버튼
- 컬럼 축소: Taxa→Tx, Specimens→Sp (헤더 title 툴팁), 폭 40px
- 저자(연도)/제목 td에 `title` 속성 추가 (마우스오버 시 전체 텍스트 툴팁)
- **버그 수정**: `year` 필드가 문자열 타입일 때 `Min()` 결과 `int` 변환 누락 → 500 에러

### 2. 저자 상세 페이지 개선 (`author_detail`)
- 논문 목록: 저자+연도 → "저자 (연도)" 병합, PDF 아이콘 컬럼 추가
- 논문 수 표시 (N건)
- `table-layout:fixed` 컬럼 폭 220px, 잘린 텍스트 title 툴팁
- 페이지네이션 추가 (20건/페이지)

### 3. 저널 상세 페이지 개선 (`journal_detail`)
- 저널명 → 영문제목/원어제목 별도 행으로 분리
- 논문 목록: 저자+제목 → "저자 (연도)" + 제목 + PDF 아이콘
- 논문 수 표시, 컬럼 폭 220px, 잘린 텍스트 title 툴팁
- 페이지네이션 추가 (20건/페이지)

### 4. 논문 상세 저널명 언어별 표시 (`reference_detail_body`)
- `reference.language == 'EN'` → 저널 영문제목 우선 (없으면 원어 fallback)
- 그 외 → 원어제목 우선 (없으면 영문 fallback)

### 5. 저널 데이터 정리 (프로덕션 DB)
- 저널 425 "고생물학회지" → 423 "한국고생물학회지" (86건)
- 저널 423 → 422 "Journal of the Paleontological Society of Korea" (108건)
- 422번 저널 통합: 413건 (영문+한글명 통합)

### 6. 스모크 테스트 추가 (56→61개)
- `test_reference_list_advanced_search`: 상세검색 파라미터 조합 7건
- `test_reference_list_sorting`: 정렬 5건 (author, title, pdf, taxa, specimens)
- `test_pdf_unavailable_list`: PDF 확보 불가 목록 페이지
- `test_author_detail_page2`, `test_journal_detail_page2`: 페이지네이션 2페이지
